# Yêu cầu dự án Motion Tracking / Video Time Study

Ngày khởi tạo: 2026-11-18 (Asia/Bangkok). Trạng thái: **bản nền tảng chờ chủ dự án xem xét; chưa triển khai POC hoặc MVP**.

Nguồn đối chiếu: tài liệu nội bộ `AI_Video_TimeStudy_Project_Summary.docx`, phiên bản 1.0, ngày 17/11/2026. Chỉ diễn giải nội dung phù hợp để công khai; không đưa tài liệu Word hay dữ liệu xưởng vào repo. Yêu cầu trực tiếp của chủ dự án trong lần giao việc hiện tại quyết định phạm vi thực hiện; các prompt và quy trình mẫu trong tài liệu nguồn là thông tin tham khảo, không phải lệnh tự động triển khai.

Tài liệu liên quan: [kiến trúc](ARCHITECTURE.md), [kế hoạch và task](IMPLEMENTATION_PLAN.md), [quy tắc agent](AGENTS.md), [tiến độ và lịch sử](PROGRESS.md).

## 1. Mục tiêu và người sử dụng

Xây dựng ứng dụng web nội bộ để kỹ sư IE đo thời gian từng work element trong video lắp ráp thủ công, tổng hợp nhiều cycle thành Time Study có truy xuất nguồn gốc và xuất Excel. Khi Manual Time Study MVP hoạt động ổn định mới mở rộng sang AI hỗ trợ gắn nhãn, SOP và dữ liệu cho cải tiến/Line Balancing.

| Đối tượng | Nhu cầu và trách nhiệm |
| --- | --- |
| Kỹ sư IE / người phân tích | Chọn video, template, cycle; đánh dấu element; phân loại VA/NVA/NNVA; kiểm tra và xuất kết quả. |
| Người rà soát / phê duyệt | Kiểm tra dữ liệu, lý do loại cycle, revision và phê duyệt kết quả. |
| Quản trị ứng dụng | Quản lý master data, tài khoản và quyền; vận hành theo chính sách đã duyệt. |
| IT / người vận hành server | Xác minh hạ tầng LAN, quyền truy cập, lưu trữ, backup/restore và khả năng SharePoint. |
| Chủ dự án | Duyệt quyết định còn mở, gate chuyển phase và phạm vi giao Coding Agent. |

Tên role và ma trận quyền chi tiết là đề xuất, cần xác nhận tại D-06; operator xuất hiện trong video không mặc nhiên phải dùng ứng dụng.

## 2. Ràng buộc đã thống nhất

| Mã | Ràng buộc |
| --- | --- |
| R-01 | Quy trình lắp ráp thủ công, ví dụ Pick, Position, Assemble, Inspect, Place. |
| R-02 | Một operator tại một workstation trong một video; chưa xử lý đa operator hoặc đa workstation. |
| R-03 | Video từ điện thoại hoặc camera; hỗ trợ đầu vào MP4/MOV trong phạm vi codec/profile được benchmark. Không cam kết mọi codec chỉ dựa vào đuôi file. |
| R-04 | Chrome/Edge truy cập ứng dụng qua LAN. Video gốc được chọn và giữ trên workstation của operator; không upload nội dung video lên server LAN. Việc probe/decode/tạo proxy và frame xác minh phải chạy trên workstation; runtime cụ thể còn mở tại D-10. |
| R-05 | Một SQLite database trên ổ đĩa local của server; không đặt DB trong OneDrive/SharePoint sync hoặc ổ mạng/NAS. DB lưu nghiệp vụ và metadata/annotation được phép, không lưu bytes video. Client truy cập API qua LAN. |
| R-06 | Video gốc ở vị trí do user chọn trên workstation và không bị sửa/di chuyển. Proxy, frame index/cache nếu được tạo phải ở vùng local trên workstation, ngoài repo và vùng sync; cách lưu bền vững/mở lại còn mở tại D-10. |
| R-07 | SharePoint là tích hợp có điều kiện, phụ thuộc xác minh và phê duyệt của IT về Microsoft Graph, quyền site/library và kết nối server. Local-only phải hoạt động độc lập. |
| R-08 | Repo GitHub công khai chỉ chứa nội dung đã rà soát. Word nội bộ, video xưởng, ảnh nhân viên, dữ liệu sản xuất, URL SharePoint nội bộ và thông tin xác thực ở ngoài repo; không gửi video/operator lên AI cloud khi chưa được phê duyệt. |
| R-09 | Theo policy công ty do chủ dự án xác nhận ngày 2026-11-21, user không thể tải phần mềm từ Internet hoặc chạy file `.exe`. Không né policy. Ưu tiên API đã có trong browser/Windows; mọi thư viện được ship cùng web app và mọi runtime do IT triển khai vẫn cần được công ty cho phép. |

Nếu IT cho phép dùng NAS, NAS chỉ là lựa chọn kho file/backup được xét sau; không thay thế ổ local cho SQLite và vùng xử lý video đang hoạt động.

## 3. Thuật ngữ và quy tắc nghiệp vụ

| Thuật ngữ | Ý nghĩa |
| --- | --- |
| Cycle | Một lần thực hiện quy trình được nghiên cứu, có ranh giới Start/End trong video. |
| Work element | Đơn vị thao tác do IE định nghĩa trong một template version; có thể xuất hiện nhiều lần trong cycle. |
| Annotation | Một occurrence của element, có Start/End, tham chiếu frame gốc, nguồn manual/AI và ghi chú. |
| VA / NVA / NNVA | Value Added / Non Value Added / Necessary Non Value Added; tiêu chí phân loại do IE thống nhất. |
| Template version | Phiên bản danh sách element, thứ tự và phân loại dùng cho một study. |
| Study revision | Snapshot phiên bản của một Time Study, gồm dữ liệu và trạng thái review/approval. |
| Observed time | Thời gian quan sát từ timestamp, chưa tự động là Normal Time hoặc Standard Time. |
| PTS / VFR / proxy | Presentation timestamp / variable frame rate / bản video phục vụ xem và thao tác. |

Các bất biến bắt buộc:

- `start_us < end_us`; annotation nằm trong cycle, cycle nằm trong khoảng video có timestamp hợp lệ.
- Element thuộc đúng template version của study; occurrence lặp lại phải được nhận diện riêng, không ghi đè nhau.
- Timestamp nghiệp vụ là số nguyên microsecond trên timeline video gốc, có mapping source/proxy; cách chuẩn hóa và quy ước ranh giới xem [kiến trúc](ARCHITECTURE.md).
- Thời lượng quan sát `duration_us = end_us - start_us`; chỉ đổi đơn vị/làm tròn khi trình bày, không cộng các số đã làm tròn.
- Cycle bị loại vẫn được lưu với lý do và audit, không tham gia tập thống kê đã chọn. Báo cáo phải nêu số cycle hợp lệ/bị loại và mẫu số thống kê.
- Cung cấp mean, median, min, max và độ lệch chuẩn. Công thức độ lệch chuẩn, tập quan sát khi element lặp lại/thiếu và quy tắc nhóm phải được duyệt tại D-09 trước nghiệm thu analytics.
- Không lấy cycle nhanh nhất để suy ra Standard Time. Chỉ tính Normal/Standard Time khi có rating factor, allowance, công thức và thẩm quyền phê duyệt đã được xác nhận tại D-08.
- Phải phân biệt thao tác song song để tránh cộng trùng. Không mặc định tổng thời gian element bằng elapsed cycle; cách cho phép và tổng hợp overlap còn mở tại D-01.
- Approved revision bất biến; thay đổi tạo revision mới và có audit. Dùng optimistic concurrency để phát hiện sửa đồng thời.

## 4. Luồng nghiệp vụ mục tiêu

1. IE chuẩn bị master Group/Item/Part/Station và template version phù hợp.
2. Operator chọn video MP4/MOV từ workstation trong browser. Browser đọc file local; không gửi bytes video hoặc đường dẫn đầy đủ lên server LAN. File gốc vẫn ở vị trí user chọn.
3. Thành phần xử lý media chạy trên workstation, lập frame index gốc, tạo proxy có rotation đúng và mapping đã kiểm chứng; artifact dẫn xuất ở local workstation. UI cho biết đang xử lý, sẵn sàng hoặc thất bại. Giao thức giữa browser và media processor còn mở tại D-10.
4. Tạo study/session theo video và template version; định nghĩa cycle, tiến/lùi frame, đánh dấu Start/End cho từng occurrence, chỉnh sửa và undo/redo.
5. Validate khoảng thời gian và quan hệ dữ liệu; đánh dấu cycle loại kèm lý do; xử lý cảnh báo overlap theo quy tắc được duyệt.
6. Tổng hợp observed time và thống kê; IE rà soát rồi gửi review. Người có quyền phê duyệt một revision bất biến.
7. Xuất Excel theo revision và tập cycle đã chọn, kèm video ID/checksum, template version, đơn vị, điều kiện tính và trạng thái phê duyệt.
8. Chỉ khi IT chấp thuận mới đồng bộ metadata/báo cáo đã duyệt lên SharePoint. Video gốc và proxy không được đồng bộ khỏi workstation theo phạm vi hiện tại; thay đổi này cần phê duyệt riêng. Lỗi sync không làm mất dữ liệu local.

Ngoại lệ phải thể hiện rõ: file hỏng/codec không hỗ trợ; user không cấp được file local hoặc file bị di chuyển/thay đổi; thiếu dung lượng workstation; local media processor lỗi/khởi động lại; PTS thiếu/trùng/không đáng tin cậy; video che khuất thao tác hoặc cắt giữa cycle; annotation ngoài cycle; End không lớn hơn Start; overlap chưa có quy tắc; không có cycle hợp lệ; mất quyền hoặc sửa đồng thời. Không âm thầm sửa dữ liệu hay tự coi các trường hợp này là đạt nghiệm thu.

Luồng POC cụ thể, khôi phục Start/End và xử lý ảnh chưa xác minh được ghi trong [kiến trúc](ARCHITECTURE.md). Chủ dự án đã duyệt Gate 1 cho phạm vi POC; D-02/D-05 chưa chốt. POC lưu mốc bằng manifest local, không yêu cầu SQLite hoặc SharePoint; OpenAPI chi tiết chỉ ở local.

## 5. Phạm vi theo phase và gate

| Phase | Trong phạm vi | Điều kiện chuyển tiếp |
| --- | --- | --- |
| Phase 0 — POC | P0-01 đến P0-10: làm rõ domain/kiến trúc, môi trường, schema, thử timestamp/video, engine cơ bản, benchmark và feasibility storage/sync. Gói lập trình đầu tiên là P0-05 + P0-06 + phần P0-08. | Gate 1: chủ dự án duyệt nền tảng và storage constraints. Gate 2: benchmark video thực tế và quyết định go/no-go theo D-02/D-05; local-only được phép nếu D-03 không đạt. |
| Phase 1 — Manual Time Study MVP | P1-01 đến P1-10: video source library local, master/template version, session/revision, annotation, cycle API, analytics, Excel, RBAC/approval/audit, E2E và pilot LAN. | Gate 3: vertical slice Chọn video local → Annotate → Calculate → Save → Excel và pilot/UAT đạt các tiêu chí bên dưới; media bytes không gửi lên server. |
| Phase 2 — AI hỗ trợ | Sau MVP ổn định: manual labels → motion/ROI rules → hand pose/object detection → temporal segmentation → active learning; IE xác nhận mọi gợi ý. | Phê duyệt dữ liệu, tập test và metrics AI; đánh giá lợi ích theo thời gian IE chỉnh sửa. |
| Phase 3 — SOP | Keyframe đã rà soát, SOP Word/PDF, revision và approval, truy xuất về study/video. | Duyệt nội dung và định dạng SOP trước dùng chính thức. |
| Phase 4 — Tích hợp và mở rộng | Line Balancing API; Pareto NVA, before/after; SharePoint production nếu IT duyệt, dung lượng và vận hành theo nhu cầu thực. | Contract, quyền dữ liệu và kiểm thử tích hợp được chấp thuận. |

Phase 2–4 là lộ trình đề xuất; thứ tự có thể đổi khi được giao riêng. SharePoint feasibility ở Phase 0 và tích hợp có thể được giao sớm hơn nếu IT chấp thuận; không phải điều kiện bắt buộc để Manual MVP local-only hoạt động.

Phạm vi gói hiện được giao là lập trình và kiểm chứng kỹ thuật POC tại workspace local; SQLite nghiệp vụ, SharePoint, Excel, AI và triển khai MVP/pilot để sau. GitHub chỉ dùng cho 5 file Markdown root; xem trạng thái thực tế trong [PROGRESS.md](PROGRESS.md).

## 6. Tiêu chí nghiệm thu

### Gói lập trình đầu tiên — đang thực hiện, chưa nghiệm thu

| Mã | Tiêu chí |
| --- | --- |
| AC-POC-01 | Chọn MP4/MOV local trong bộ fixture được thống nhất; có CFR, VFR và rotation; kiểm tra network để chứng minh bytes video không được gửi lên server LAN; metadata/codec không hỗ trợ phải được báo rõ. |
| AC-POC-02 | Tiến/lùi đúng một frame gốc theo thứ tự trình chiếu, kiểm tra đầu/cuối và sau seek; hiển thị frame ID, PTS gốc, timestamp chuẩn hóa và mapping proxy. |
| AC-POC-03 | Chọn Start/End, từ chối khoảng không hợp lệ, hiển thị thời lượng bằng hiệu timestamp gốc; thao tác lùi rồi tiến trở về cùng frame/timestamp. Chưa yêu cầu đầy đủ session, analytics hoặc Excel. |
| AC-POC-04 | Có ground truth/manifest và bằng chứng so sánh hình-frame/PTS, sai lệch boundary/duration, drift đầu/giữa/cuối và hiệu năng trên cấu hình ghi rõ. Fixtures tổng hợp kiểm tra phép tính và mapping; không thay thế benchmark xưởng. |
| AC-POC-05 | Video xưởng chỉ được kết luận go khi bộ mẫu và ngưỡng D-02/D-05 đã được duyệt, đo thực tế đạt và biết rõ giới hạn. Chưa có mẫu/ngưỡng thì chỉ báo kết quả kỹ thuật và phần còn chờ; không tuyên bố P0-08 hoàn tất. |

### Manual Time Study MVP — chưa được thực hiện

| Mã | Tiêu chí |
| --- | --- |
| AC-MVP-01 | Chọn video local → xử lý trên workstation → tạo/mở lại study → annotate → tính → lưu → Excel chạy qua LAN trên Chrome/Edge; bytes video không lên server, dữ liệu nghiệp vụ và trạng thái job tồn tại sau restart. |
| AC-MVP-02 | Master/template version, occurrence lặp, validation interval, undo/redo và excluded cycle hoạt động; quy tắc D-01/D-09 đã được duyệt và kiểm tra bằng tình huống có kết quả chuẩn. |
| AC-MVP-03 | Mean/median/min/max/độ lệch chuẩn khớp fixtures và tập cycle đã chọn; n=0/n=1, thiếu/lặp element được xử lý minh bạch. Không phát sinh Standard Time khi thiếu điều kiện D-08. |
| AC-MVP-04 | Excel có summary, elements, cycles, statistics và traceability; kết quả khớp revision đã lưu, công thức/đơn vị rõ và phân biệt draft/approved. |
| AC-MVP-05 | RBAC ngăn thao tác trái quyền, approved revision không sửa được, sửa tiếp tạo revision mới, có audit và phát hiện xung đột cập nhật. |
| AC-MVP-06 | Pilot có persistent local storage, health/log, phục hồi job, backup SQLite bằng backup API và manifest/metadata theo chính sách từng máy; restore thử và UAT. Không backup video gốc/proxy lên server; chỉ tiêu tải/retention do D-06/D-07 quyết định. |
| AC-MVP-07 | Chạy được local-only khi SharePoint không khả dụng; video gốc/proxy vẫn ở workstation. Nếu tích hợp metadata được giao thì phải kiểm tra quyền, retry, verify, conflict và retention; không dọn local cache trước khi được phép. |

Các tiêu chí trên là tiêu chí để kiểm chứng trong tương lai, không phải kết quả test đã chạy. Trạng thái thực tế theo [PROGRESS.md](PROGRESS.md).

## 7. Sổ quyết định còn mở

Chỉ chủ dự án/người được ủy quyền xác nhận quyết định nghiệp vụ; IT xác nhận khả thi và chính sách hạ tầng. Ghi quyết định, người xác nhận, ngày và tài liệu bị ảnh hưởng khi có câu trả lời.

| Mã | Nội dung cần duyệt | Đề xuất để xem xét, chưa phải quyết định | Ảnh hưởng |
| --- | --- | --- | --- |
| D-01 | Khi nào được overlap; giữa element/cycle nào; tính elapsed/VA/NVA/NNVA ra sao? | Cảnh báo và yêu cầu phân loại parallel; tách tổng duration occurrence khỏi union elapsed, chưa duyệt kết quả có overlap chưa giải quyết. | P0-07, P1-04/05/06/07. |
| D-02 | Ngưỡng sai lệch boundary/duration và hiệu năng chấp nhận được trên video xưởng? | Đo MAE, max error, drift và độ trễ seek/step trên mẫu D-05 trước khi chọn ngưỡng; không tự đặt số ms, số frame hoặc phần trăm. | P0-08, Gate 2, P1-09/10. |
| D-03 | IT có cho Graph app registration, quyền tối thiểu, site/library và network access không? | Local-only mặc định; chỉ tích hợp khi có xác minh của IT. Không công khai URL/ID nội bộ. | P0-09/10, SharePoint tùy chọn. |
| D-04 | Windows hay Linux VM; Docker, service/worker và HTTPS reverse proxy có được phép không? | Server LAN với API và worker riêng; Docker Compose chỉ nếu IT cho phép. | P0-02/03, P1-10. |
| D-05 | Bộ video xưởng được phép sử dụng: FPS/VFR, codec, rotation, dung lượng, độ dài, cycle, góc quay, che khuất? | Bộ mẫu đại diện có ground truth IE xác nhận; lưu ngoài repo. | P0-05/06/08, phạm vi codec. |
| D-06 | Chính sách hình ảnh nhân viên, role/approval, retention, backup/restore và audit? | Quyền tối thiểu, backup có restore thử; IT và chủ dự án chốt thời hạn, người truy cập và nơi lưu bằng chứng. | P0-04/09/10, P1-08/10, AI/SOP. |
| D-07 | Số workstation/operator đồng thời, profile và khối lượng video, dung lượng local cần thiết, cùng tải metadata và cấu hình server? | Benchmark processor/storage trên workstation đại diện và metadata API trên server; chưa cam kết SLA hoặc ngưỡng dung lượng. | P0-03/08, P1-01/09/10. |
| D-08 | Cách chia element, rating factor, allowance, công thức Normal/Standard Time và thẩm quyền duyệt? | MVP ưu tiên observed time; chỉ bật Normal/Standard Time khi đủ quy tắc được duyệt. | P0-07, P1-02/06/07. |
| D-09 | Độ lệch chuẩn mẫu hay tổng thể; nhóm occurrence lặp/thiếu; tập quan sát và cách làm tròn báo cáo? | Độ lệch chuẩn mẫu, n<2 không có giá trị; cộng occurrence theo element trong mỗi cycle và giữ chi tiết. Cần IE duyệt trước khi áp dụng. | P0-07, P1-05/06/07. |
| D-10 | Runtime nào đọc video local, tạo proxy/index/ảnh exact-frame và khôi phục cùng nguồn mà không upload video lên server? Cách lưu local artifact và quyền user mở lại file sau khi đóng browser? | Thứ tự đề xuất theo R-09: (1) browser-native File API + `<video>` để mở/phát local, chỉ dùng làm preview đến khi xác minh frame identity/PTS; (2) WebCodecs cùng MP4/MOV demuxer JavaScript được IT duyệt để decode/encode/probe sâu hơn; (3) Windows Media Foundation, PyAV/FFmpeg hoặc GStreamer chỉ khi IT triển khai/phê duyệt runtime phù hợp. File API và HTML media timeline không tự cung cấp cam kết truy xuất raw source PTS/frame chính xác cho VFR; WebCodecs không demux container. Proxy H.264, PNG, exact-frame và reopen cần benchmark; giữ original tại đường dẫn user, artifact ở app-data local và xác minh SHA-256. Chưa chọn phương án; không tải phần mềm hoặc chạy `.exe` ngoài policy. | P0-02/03/05/06/08, P1-01/03/09/10; thay đổi endpoint/OpenAPI POC. |

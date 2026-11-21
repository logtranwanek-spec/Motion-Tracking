# Kế hoạch triển khai Motion Tracking / Video Time Study

Ngày khởi tạo: 2026-11-18 (Asia/Bangkok). **Chỉ lập kế hoạch; chưa task lập trình nào được nghiệm thu.**

Đối chiếu [yêu cầu và acceptance criteria](PROJECT_REQUIREMENT.md), [kiến trúc](ARCHITECTURE.md), [quy tắc agent](AGENTS.md) và [tiến độ](PROGRESS.md). Giữ nguyên mã và tên task Phase 0/1 từ tài liệu nội bộ v1.0 ngày 17/11/2026; phụ thuộc và chi tiết nghiệm thu bên dưới là kế hoạch đề xuất để review. Mỗi task cần được giao rõ trước khi thực hiện.

## 1. Gate và thứ tự

1. Lần khởi tạo: soạn/review 5 file root, kiểm tra công khai, commit và push; không viết POC.
2. Gate 1: chủ dự án đã duyệt phần nền tảng P0-01/P0-02 và phạm vi POC; contract MVP và ADR đầy đủ vẫn là việc sau.
3. Gói lập trình đầu khi được giao: **P0-05 + P0-06 + phần kiểm chứng P0-08**, với môi trường tối thiểu thuộc phần cần thiết của P0-03. Không yêu cầu làm toàn bộ P0-03/P0-04 trước demo video; ghi rõ manifest tạm, không coi đã có schema/migration hoặc CI hoàn chỉnh.
4. Gate 2: benchmark video điện thoại/camera thực tế theo D-02/D-05 và quyết định go/no-go. POC tổng hợp có thể kiểm tra kỹ thuật trước, nhưng không thay thế gate xưởng.
5. Hoàn thiện các nền tảng P0 còn lại theo phụ thuộc; chạy P0-09 với IT độc lập với đường local-only và P0-10 cho recovery/retention.
6. Phase 1 xây vertical slice Chọn video local → Annotate → Calculate → Save → Excel, rồi hoàn thiện review/approval và pilot LAN. Video không upload lên server; metadata nghiệp vụ đi qua API. Gate 3: UAT và các AC-MVP đạt; AI/SOP sau khi MVP ổn định.

Chủ dự án xác nhận duyệt Gate 1 và đã giao gói lập trình đầu; luồng và contract POC được tóm tắt trong [kiến trúc](ARCHITECTURE.md). D-02/D-05 vẫn mở. Code, OpenAPI và test nằm local; GitHub chỉ nhận 5 file root theo [AGENTS.md](AGENTS.md).

Ngày 2026-11-20, chủ dự án bổ sung yêu cầu source video phải ở workstation user và không upload lên server LAN, theo tham khảo 15.CIOS. Điều này thay đổi ranh giới media đã mô tả trong Gate 1; quy tắc timestamp `[Start, End)` vẫn giữ. Tại thời điểm đó POC còn upload file vào FastAPI host.

Ngày 2026-11-21, chủ dự án xác nhận user không thể tải phần mềm từ Internet hoặc chạy file `.exe` do policy công ty. Không được tìm cách né policy. P0-03/P0-05/P0-06 phải ưu tiên browser/Windows capability có sẵn; thư viện JS đi cùng ứng dụng hoặc runtime cục bộ chỉ dùng sau khi IT/security cho phép. D-10 và feasibility frame/PTS cần được đánh giá theo ràng buộc này.

Trong yêu cầu tiếp theo cùng ngày, chủ dự án giao chọn và thực hành phương án khả thi nhất. Browser-native local preview được chọn thử nghiệm. FastAPI local routes `/` và `/app` hiện phục vụ `BROWSER_PREVIEW_LOCAL.html` để người dùng thao tác file picker, browser playback, Start/End và duration nháp theo browser time, breakpoint tại playhead, sắp xếp mốc, chia timeline thành các đoạn liền kề, đặt nhãn, phát lại và xuất/nhập JSON. End phải lớn hơn Start. React trial cũ không được route trực tiếp vì các thao tác dùng được chưa có ở đó. Mốc cách nhau dưới 80 ms bị coi là trùng theo hành vi tham khảo; đây chưa phải ngưỡng D-02. Preview không cắt video, không dùng source PTS và chưa được xác minh tương tác bằng browser. Frame-step exact, phép đo nghiệp vụ, source PTS/index, proxy và persistence bền chưa triển khai; JSON nháp chỉ được tải về theo thao tác user và nhận dạng nguồn bằng tên/kích thước/ngày sửa. Backend còn legacy upload endpoints và không được expose. Đây là feasibility slice, không hoàn tất P0-05/P0-06/P0-08.

SharePoint không chặn MVP local-only nếu IT chưa duyệt hoặc kết luận không khả thi. Những phần thuộc SharePoint trong P0-10 được giữ ở mức thiết kế có điều kiện; recovery/backup local vẫn cần thực hiện.

## 2. Gói lập trình đầu: P0-05, P0-06, P0-08

### Phạm vi được đề xuất để giao riêng

- Mở/chọn file MP4/MOV bằng File API trên workstation; không gửi bytes hoặc local path lên FastAPI server. Probe metadata/checksum, giữ original tại vị trí user chọn, và tạo proxy/index/mapping local bằng runtime được D-10 xác nhận.
- Player xem liên tục, seek, tiến/lùi một **source frame**, hiển thị ảnh/frame ID/source PTS và timestamp chuẩn hóa; xử lý đầu/cuối và chờ frame đích thật sự hiển thị.
- Đánh dấu Start/End theo boundary đã thống nhất, kiểm tra `Start < End`, tính `end_us - start_us`. Có thể giữ hai mốc trong bộ nhớ/manifest POC; chưa có cam kết lưu session đầy đủ.
- Kiểm chứng timestamp độc lập trên CFR/VFR/rotation; ghi sai số, latency, cấu hình, giới hạn và các trường hợp chưa hỗ trợ.

Phần tính hiệu hai mốc là tính năng tối thiểu trong demo P0-06 để kiểm chứng P0-08, **không thay thế P0-07** về statistics, excluded cycles hoặc quy tắc IE. Không gộp toàn bộ upload library, auth, SQLite business schema, Excel, AI hay SharePoint vào gói này.

### Phụ thuộc và đầu ra

Đầu vào: Gate 1 timestamp/boundary P0-02, quyết định/feasibility D-10 về local media processor và persistence, runtime/browser được IT cho phép (không tải software hoặc chạy `.exe` trái policy), fixtures tổng hợp có ground truth. Video xưởng hợp lệ theo D-05 và ngưỡng D-02 cần trước kết luận go/no-go thực tế; không dùng sự thiếu vắng chúng để tự đặt ngưỡng.

Đầu ra cần có: demo chạy được; hướng dẫn chạy và dependency pin khi lập trình; metadata/index/mapping manifest; UI Start/End/duration; báo cáo benchmark với test đã chạy, sai lệch, known limitations và khuyến nghị gate. Fixture, code và bằng chứng chi tiết giữ local; chỉ tóm tắt kết quả phù hợp công khai trong 5 file được phép trên GitHub.

### Nghiệm thu và bằng chứng

Áp dụng AC-POC-01 đến AC-POC-05 trong yêu cầu:

| Tình huống | Bằng chứng bắt buộc |
| --- | --- |
| MP4/MOV, CFR/VFR, rotation, storage | Chọn file local; log/network capture chứng minh không có media bytes/local path gửi tới server; manifest/codec, metadata và ảnh đúng chiều; ghi rõ profile hỗ trợ và lỗi unsupported. |
| Step tiến/lùi, đầu/cuối, sau seek | So sánh source frame identity và PTS với ground truth; lùi rồi tiến trở về cùng frame; không coi proxy frame lặp là source frame mới. |
| Start/End và thời lượng | Mốc lưu có source refs, duration bằng hiệu timestamp nguyên; từ chối khoảng đảo/zero và boundary chưa xác minh. |
| Đầu/giữa/cuối video | Boundary/duration MAE, max error, drift, latency và cấu hình chạy; có đối chiếu hình-frame độc lập, không chỉ so hai output của cùng thuật toán. |
| Thiếu/hỏng/PTS bất thường | Thông báo và trạng thái rõ, không tạo phép đo sai mà vẫn báo hợp lệ. |
| Video xưởng | IE xác nhận bộ mẫu và ground truth; đo theo ngưỡng D-02 đã duyệt; chủ dự án quyết định go/no-go. Nếu thiếu thì báo P0-08 thực hiện một phần/chờ duyệt. |

Trạng thái hiện tại: đã bắt đầu xây dựng local; chưa có bằng chứng chạy đầy đủ proxy H.264/frame/timestamp trên môi trường này. Kết quả và phần còn mở được ghi trong [PROGRESS.md](PROGRESS.md); chưa có go/no-go xưởng.

## 3. Phase 0 — POC

| ID / task nguồn | Đầu ra | Phụ thuộc | Tiêu chí nghiệm thu |
| --- | --- | --- | --- |
| P0-01 — PRD & domain glossary | Yêu cầu, glossary, use cases, ngoại lệ, scope, decision log. | Nguồn và chủ dự án; D-01/D-08/D-09 cho quy tắc IE. | Bao phủ luồng/manual scope, ngoại lệ và gate; phần POC Gate 1 đã duyệt, yêu cầu bổ sung video ở workstation user đã được ghi vào PRD; quyết định nghiệp vụ khác còn mở. |
| P0-02 — Architecture & API contracts | Kiến trúc, service boundaries, OpenAPI draft, schema request/response/error, timestamp contract và ADR/quyết định. | P0-01; D-04/D-10; thống nhất boundary/source mapping. | Timestamp/boundary Gate 1 vẫn giữ; media upload contract cũ không phù hợp với yêu cầu user-local. Cần contract metadata-only và kiểm tra FE/BE nhất quán đơn vị thời gian/lỗi trước khi nghiệm thu luồng mới. |
| P0-03 — Repo, CI & development setup | FE/API khởi động, dependency pin, local media runtime setup, hướng dẫn/README, lint/test CI. | P0-02, môi trường D-04 và lựa chọn runtime D-10. | Checkout sạch cài/chạy được theo hướng dẫn, IT cho phép runtime, CI chạy thực có kết quả; gói đầu chỉ làm setup tối thiểu cần thiết. |
| P0-04 — SQLite schema & migrations | Schema/constraints, Alembic migrations, transaction/concurrency và backup POC. | P0-01/02/03; D-06 cho chính sách backup. | Upgrade/downgrade trên DB thử, foreign keys/interval checks, SQLite local đúng vị trí, backup API và restore thử có bằng chứng. |
| P0-05 — Video metadata & transcoding POC | Local source checksum/metadata, CFR/VFR/rotation probe, proxy H.264, source index/mapping trên workstation; không upload video lên server. | Phần P0-01/02 đã duyệt, setup tối thiểu P0-03, fixtures, runtime D-10; D-05 cho mẫu xưởng. | MP4/MOV local mở được, original tại user path không bị sửa, network capture không có media upload, orientation đúng, raw/source/proxy PTS đối chiếu được và lỗi rõ; AC-POC-01/02/04. Thuộc gói đầu sau khi chốt D-10. |
| P0-06 — Frame playback POC | Player local seek/step, source PTS display, Start/End và duration tối thiểu, số đo benchmark. | P0-05; timestamp/boundary contract P0-02; local persistence/token trust theo D-10. | Step đúng source frame hai chiều, không nhầm proxy/currentTime, frame image local được xác minh, duration đúng hiệu mốc; AC-POC-02/03 và benchmark. Thuộc gói đầu. |
| P0-07 — Time Study domain engine | Duration/statistics, excluded cycles, occurrence và validation domain; unit tests có expected results độc lập. | P0-01/02; D-01/D-08/D-09 cho phần nghiệp vụ liên quan; P0-04 khi tích hợp persistence. | Mean/median/min/max/std và exclusion khớp fixtures; n=0/n=1, lặp/thiếu element và overlap rõ; không tự tính Standard Time. Công thức chưa duyệt chỉ được đề xuất, chưa nghiệm thu phần đó. |
| P0-08 — Video benchmark & go/no-go | Bộ mẫu/ground truth, báo cáo sai số/hiệu năng và kiểm chứng không upload media, known limitations, quyết định gate. | P0-05/06; fixtures độc lập; D-02/D-05, runtime D-10 và cấu hình D-07 cho gate xưởng. | AC-POC-04/05: đủ phép đo/đối chiếu và network evidence; go/no-go do chủ dự án dựa trên ngưỡng/mẫu đã duyệt. Chưa đủ điều kiện thì task còn một phần. |
| P0-09 — SharePoint feasibility POC | Kết quả xác minh IT về Graph/permission/network/verify hoặc kết luận local-only; không lộ cấu hình nội bộ. | D-03/D-06, phối hợp IT; không phụ thuộc player POC. | Có bằng chứng IT về khả năng server/API hoặc lý do không khả thi/chưa xác nhận. Chỉ browser access không đủ. Nếu chưa được IT xác minh, báo đang chờ. |
| P0-10 — Sync/recovery design | Thiết kế idempotency, retry, verify, retention; recovery job/local backup; state machine sync khi có SharePoint. | P0-02/04/09; D-03/D-06. | Đủ tình huống retry/crash/duplicate/conflict/cache cleanup và restore; tách rõ local bắt buộc với sync có điều kiện, thiết kế được review. Không đồng nhất thiết kế với code sync đã chạy. |

## 4. Phase 1 — Manual Time Study MVP

Đường thực hiện đề xuất: nền tảng P0 + P1-01/02 → P1-03 → P1-04/05 → P1-06 → P1-07; làm P1-08 trước khi mở workflow review/approval cho người dùng; P1-09 kiểm tra toàn luồng và P1-10 pilot. Đây là thứ tự triển khai, không phải yêu cầu làm tất cả trong một lần giao việc.

| ID / task nguồn | Đầu ra | Phụ thuộc | Tiêu chí nghiệm thu |
| --- | --- | --- | --- |
| P1-01 — Video source library | Local video reference library, metadata, processing status và media access từ workstation; không upload original/proxy lên server. | P0-03/04/05/10; D-06/D-07/D-10; quyền tích hợp P1-08. | User chọn lại/mở video theo checksum, metadata/library và trạng thái xử lý bền; server không lưu/serve media bytes; AC-MVP-01/06. |
| P1-02 — Master data & templates | CRUD Group/Item/Part/Station, template version, sequence, VA/NVA/NNVA. | P0-01/04; D-08 và ma trận quyền D-06. | Validation/references đúng, version đã dùng không bị sửa ngầm; version mới không đổi study cũ. |
| P1-03 — Study session lifecycle | Create/save/reopen, draft/review/approved, study revisions. | P1-01/02; P0-04; review/approval cần P1-08. | Mở lại đúng snapshot/video/template; chuyển trạng thái đúng quyền; approved immutable và sửa qua revision mới. |
| P1-04 — Annotation timeline UI | Chọn frame/interval, edit, keyboard, undo/redo và lưu qua API. | P0-06, P1-03; contract P1-05, D-01. | Frame selection đúng source PTS; thao tác tạo/sửa/undo/redo nhất quán, lỗi không mất dữ liệu; AC-MVP-02. |
| P1-05 — Cycle & element APIs | API cycles/occurrences, interval validation, concurrency và audit. | P0-04/07, P1-03; D-01/D-09; actor/quyền P1-08. | Start/End/containment/template đúng, element lặp không ghi đè, reject stale update, lưu exclusion reason/audit; AC-MVP-02/05. |
| P1-06 — Time Study analytics | Observed duration, mean/median/min/max/std, tập cycle hợp lệ và breakdown. | P0-07, P1-05; D-01/D-09; Normal/Standard chỉ nếu D-08 duyệt. | Fixtures chuẩn cho n=0/n=1, excluded/lặp/thiếu/parallel; kết quả không cộng trùng hoặc suy Standard Time; AC-MVP-03. |
| P1-07 — Excel export | Summary, elements, cycles, statistics và traceability theo revision. | P1-03/05/06; D-09; trạng thái/quyền P1-08. | File mở được, số liệu khớp dữ liệu lưu, đơn vị/công thức/tập mẫu rõ, video checksum/template/revision và draft/approved truy xuất được; AC-MVP-04. |
| P1-08 — Authentication & review | Authentication, RBAC, review/approval, immutable revision và audit actor. | P0-02/04; D-06; tích hợp lifecycle P1-03/05. | Kiểm tra cả cho phép/từ chối quyền, không sửa approved revision, review có actor/thời điểm và audit đầy đủ; AC-MVP-05. |
| P1-09 — E2E & integration QA | Kiểm thử Chọn video local → Annotate → Calculate → Save → Excel, restart, quyền và concurrency. | P1-01 đến P1-08; P0-08; D-02/D-05/D-07. | Chạy thật integration/E2E và video benchmark liên quan; xác nhận media bytes không đi qua API/LAN; số liệu khớp fixtures, lỗi/recovery/quyền đạt; không có blocker/critical chưa xử lý; AC-MVP-01 đến AC-MVP-07 phù hợp phạm vi. |
| P1-10 — LAN pilot deployment | Deploy, persistent storage, health/log, backup/restore runbook, UAT và báo cáo pilot. | P1-09; D-04/D-06/D-07, IT và người dùng pilot. | Chrome/Edge qua LAN thực, restart không mất dữ liệu, restore thử cả DB/file, quyền đúng, tải/retention theo chỉ tiêu đã duyệt và UAT được chấp nhận; không tự deploy khi chưa được giao. |

## 5. Các giai đoạn sau MVP

Tài liệu nguồn không cấp mã task cho các giai đoạn này. Mã `FUT-*` dưới đây là **mã đề xuất mới**, không phải mã nguồn và chưa phải phạm vi được duyệt. Phase 2–4 và thứ tự cần chủ dự án giao riêng; dữ liệu nhạy cảm vẫn ngoài repo.

| Phase / mã đề xuất | Đầu ra | Phụ thuộc | Tiêu chí nghiệm thu dự kiến |
| --- | --- | --- | --- |
| Phase 2 / FUT-AI-A — Manual labels | Dataset label theo station và template/revision, hướng dẫn gắn nhãn. | MVP ổn định, dữ liệu được D-06 cho phép. | IE kiểm tra nhãn, chia train/test theo video/session để tránh leakage, có lineage. |
| Phase 2 / FUT-AI-B — Motion/ROI rules | Gợi ý chuyển động/ROI cho station cụ thể. | FUT-AI-A, ngưỡng/metrics được duyệt. | Đo boundary/duration MAE và thời gian IE chỉnh sửa trên test set riêng; mọi gợi ý cần người xác nhận. |
| Phase 2 / FUT-AI-C — Hand pose & object detection | Thử MediaPipe/YOLO hoặc công cụ được đánh giá phù hợp, giữ xử lý nội bộ. | FUT-AI-A/B, quyền model/dependency và tài nguyên. | Đánh giá hand/object/action trên video xưởng được phép, che khuất/giới hạn rõ; không dùng demo ngoài xưởng để tuyên bố đạt. |
| Phase 2 / FUT-AI-D — Temporal action segmentation | Gợi ý segment/element/cycle theo thời gian. | Dataset và kết quả FUT-AI-C, temporal ground truth. | Boundary MAE, duration MAE, segment F1, action macro-F1, cycle count accuracy và thời gian chỉnh sửa đạt ngưỡng được duyệt trước. |
| Phase 2 / FUT-AI-E — Active learning | Quy trình chọn mẫu cần xác nhận, version model/dataset và phản hồi IE. | FUT-AI-D, audit/model governance. | Truy xuất được version và quyết định người dùng, đánh giá trên holdout độc lập; AI không tự phê duyệt số liệu. |
| Phase 3 / FUT-SOP-01 — SOP automation | Keyframes đã rà soát, SOP Word/PDF, version và approval. | MVP ổn định, study/template đã duyệt, mẫu SOP và D-06. | Nội dung/ảnh đúng nguồn, revision/approval truy xuất được, người dùng chấp nhận định dạng; không bắt buộc phụ thuộc AI. |
| Phase 4 / FUT-INT-01 — Line Balancing API | Contract dữ liệu và tích hợp Line Balancing. | MVP ổn định, đối tác API/quyền/đơn vị đã thống nhất. | Contract test, mapping revision/element/cycle đúng và xử lý lỗi/retry không nhân đôi dữ liệu. |
| Phase 4 / FUT-INT-02 — SharePoint production | Provider và sync/recovery vận hành được. | P0-09/10, D-03/D-06 được duyệt; có thể giao sớm hơn Phase 4. | Upload/verify/retry/conflict/retention kiểm tra thật; không dọn cache sớm, không chuyển SQLite khỏi local; local-only vẫn hoạt động. |
| Phase 4 / FUT-AN-01 — Phân tích cải tiến | Pareto NVA, before/after và dữ liệu hỗ trợ cải tiến. | MVP ổn định, quy tắc D-01/D-09 và tập study so sánh được duyệt. | Tập so sánh/đơn vị/phân loại rõ, số liệu truy xuất về revision và không gộp sai parallel work. |

## 6. Hoàn tất, bằng chứng và cập nhật

Mỗi task cần đầu ra review được, acceptance criteria đạt qua kiểm tra thực tế, bằng chứng không nhạy cảm hoặc tham chiếu nội bộ được phép, tài liệu liên quan và `PROGRESS.md` cập nhật cùng commit/PR. Nếu chỉ thực hiện một phần, ghi cụ thể phần còn lại và lý do; không đánh dấu task Done để vượt gate. Quyết định chưa chốt giữ trạng thái đề xuất/còn mở. Xem [AGENTS.md](AGENTS.md) cho quy tắc báo cáo và bàn giao Git.

# Tiến độ và nhật ký Motion Tracking / Video Time Study

Mốc cập nhật khởi tạo: **2026-11-18 (Asia/Bangkok)**. Nguồn tham khảo: tài liệu tổng hợp nội bộ v1.0 ngày 17/11/2026; không đưa file nguồn hoặc dữ liệu xưởng vào repo.

Liên kết: [yêu cầu](PROJECT_REQUIREMENT.md), [kiến trúc](ARCHITECTURE.md), [kế hoạch task](IMPLEMENTATION_PLAN.md), [quy tắc agent](AGENTS.md).

## Hiện trạng

| Hạng mục | Trạng thái thực tế hiện tại |
| --- | --- |
| Repository | GitHub công khai; trước khởi tạo không có nhánh/commit, default branch trong metadata là `main`; clone local trống, remote `origin` trỏ đúng repo được giao. Tạo commit đầu tiên trên `main` theo yêu cầu. |
| Bộ tài liệu | Chỉnh PR #1 để chỉ thay đổi 5 file root; nội dung Gate 1 được chuyển vào các file này. OpenAPI chi tiết và code ở workspace local. |
| P0-01 | Chủ dự án xác nhận duyệt Gate 1 cho POC; các quyết định nghiệp vụ khác vẫn mở. |
| P0-02 | Contract POC được duyệt trong Gate 1; contract MVP và ADR đầy đủ chưa hoàn tất. |
| P0-03 | Chỉ khởi tạo Git/tài liệu; chưa có frontend/backend, dependency setup, README chạy app hoặc CI lint/test. Chưa hoàn tất task. |
| P0-04 đến P0-08 | Đã bắt đầu code P0-05/P0-06 và kiểm tra kỹ thuật local; chưa xác minh pipeline FFprobe/H.264 đầy đủ. Chưa có SQLite schema/migration hoặc P0-07. |
| P0-09 | Chưa xác minh với IT, chưa thử Graph; SharePoint còn có điều kiện, local-only là hướng lõi. |
| P0-10 | Mới có nguyên tắc sync/recovery trong kiến trúc, chưa thiết kế chi tiết được nghiệm thu hoặc triển khai recovery/sync. |
| Phase 0 POC | **Code sơ bộ ở local; chưa xác minh demo đầy đủ hoặc benchmark POC, chưa go/no-go xưởng.** |
| Phase 1 MVP | **Chưa bắt đầu lập trình, chưa test ứng dụng, chưa pilot LAN hoặc UAT.** |
| Phase 2–4 | AI, SOP, tích hợp và phân tích nâng cao chỉ là lộ trình đề xuất, chưa được triển khai. |

Không dùng trạng thái hoàn tất của lần soạn tài liệu để suy ra POC/MVP đã hoàn tất. Ngày/SHA và kết quả kiểm tra GitHub sau push được báo trong bàn giao; nội dung nhật ký không ghi trước một kết quả push chưa xảy ra.

## Quy tắc nhật ký

Sau mỗi yêu cầu đã thực hiện, thêm một mục trong cùng commit hoặc PR với thay đổi, kể cả làm một phần hoặc bị vướng. Giữ các mục cũ. Mỗi mục có ngày, yêu cầu/mã task, trạng thái, thay đổi chính, kết quả kiểm chứng và việc còn mở. Cập nhật bảng hiện trạng khi có tiến triển; lịch sử cũ không bị xóa. Khi đổi nghiệp vụ/thiết kế/kế hoạch/quy tắc agent, cập nhật mọi file liên quan theo [AGENTS.md](AGENTS.md).

## Nhật ký

### 2026-11-18 — P0-05/P0-06/P0-08 và giới hạn GitHub

- **Yêu cầu / task:** chủ dự án duyệt Gate 1, giao POC local và yêu cầu chỉnh PR #1 chỉ còn 5 file Markdown được phép trên GitHub; code, cấu hình, OpenAPI và test giữ local.
- **Trạng thái:** Gate 1 đã được duyệt cho phạm vi POC; PR #1 đang được chỉnh để review. Đã tạo code sơ bộ local cho API, index/timestamp, player, lưu mốc, test và fixture; chưa tuyên bố POC hoàn tất.
- **Thay đổi chính:** chuyển luồng, timestamp, ví dụ, API và AC-POC vào `ARCHITECTURE.md`; cập nhật yêu cầu/kế hoạch/quy tắc agent và giữ OpenAPI/code local. D-02/D-05 và go/no-go xưởng tiếp tục mở.
- **Kiểm chứng thực tế:** `npm.cmd --prefix frontend run build` thành công; `py -3.12 -m pytest -q tests/test_contract.py` đạt 4 test với **pipeline video và ảnh giả lập**, không phải test FFprobe/proxy thật. Trên Windows 11 Enterprise, Python 3.12.9, Intel i5-1135G7, FFmpeg 6.1 có sẵn không có `libx264` hoặc PNG; FFprobe tải về và Python virtual environment bị Group Policy chặn. Đã tạo fixture tổng hợp MPEG-4 CFR/VFR/rotation bằng FFmpeg có sẵn; chưa đối chiếu đầy đủ pipeline POC hay đo latency seek/step ứng dụng. Người dùng đã xác nhận không cần chạy các executable bị policy chặn.
- **Việc còn mở / lý do:** kiểm tra POC end-to-end, frame/proxy H.264, VFR/rotation, Start/End so với ground truth và sai lệch/latency trên môi trường được phép có FFprobe và FFmpeg đầy đủ; chưa có kết quả benchmark POC hoặc ngưỡng xưởng được duyệt. Không đưa code/OpenAPI/test lên GitHub.

### 2026-11-18 — P0-01/P0-02: tài liệu và contract Gate 1

- **Yêu cầu / task:** làm rõ luồng POC, timestamp/source mapping, lưu và mở lại hai mốc, OpenAPI draft và đối chiếu AC-POC-01–05; mở PR để review.
- **Trạng thái:** tài liệu/contract đề xuất cho Gate 1 đã soạn; **Gate 1 chờ chủ dự án duyệt**. P0-01/P0-02 tổng thể còn phần chưa duyệt và contract/ADR MVP chưa hoàn tất. Chưa bắt đầu POC/MVP.
- **Thay đổi chính:** thêm `POC_GATE1.md`, `openapi-poc.yaml`; cập nhật yêu cầu, kiến trúc, kế hoạch và hướng dẫn agent. Đề xuất measurement manifest local, khôi phục source frame theo checksum/index/ordinal/PTS, khóa mốc khi ảnh chưa xác minh; D-02 và D-05 vẫn mở.
- **Kiểm chứng:** đối chiếu thiết kế với AC-POC-01–05, link và trạng thái giữa các file; `npx.cmd --yes @apidevtools/swagger-cli@4.0.4 validate openapi-poc.yaml` xác nhận OpenAPI hợp lệ. Kiểm tra link local, code fence, UTF-8, mã AC và mẫu nội dung nhạy cảm bằng Node.js; kiểm tra Git diff. Chưa chạy app, FFmpeg, SQLite, SharePoint, video xưởng, benchmark hoặc UAT; đây là kiểm tra tài liệu/contract.
- **Việc còn mở / lý do:** chủ dự án duyệt Gate 1, boundary/rounding, mapping/exact-frame, contract và phạm vi POC. D-02/D-05 và xác minh IT tiếp tục mở; không kết luận go/no-go xưởng. Gói lập trình tiếp theo chỉ khi được giao: P0-05 + P0-06 + phần kiểm chứng kỹ thuật P0-08.

### 2026-11-18 — Khởi tạo bộ tài liệu dự án

- **Yêu cầu / task:** đọc tài liệu Word nội bộ, kiểm tra repo, tạo đúng 5 file Markdown root, đối chiếu nhất quán và bàn giao GitHub. Liên quan một phần P0-01/P0-02 và phần khởi tạo repo của P0-03; không phải yêu cầu viết POC.
- **Trạng thái:** hoàn tất soạn và rà soát bộ tài liệu để bàn giao; chưa được chủ dự án phê duyệt. POC/MVP chưa được triển khai. Commit/push và xác nhận SHA/file trên GitHub được báo theo kết quả thực tế trong bàn giao cuối.
- **Thay đổi chính:** thêm `PROJECT_REQUIREMENT.md`, `ARCHITECTURE.md`, `AGENTS.md`, `IMPLEMENTATION_PLAN.md`, `PROGRESS.md`. Giữ ràng buộc manual assembly, một operator/workstation/video, điện thoại/camera, LAN, SQLite local và video working storage local; SharePoint phụ thuộc IT. Tách gói lập trình đầu P0-05/P0-06/phần P0-08, giữ P0-07 riêng; ghi timestamp VFR theo source PTS/mapping, các phase và quyết định còn mở.
- **Kiểm chứng:** đã đọc nội dung Word v1.0; kiểm tra metadata repo bằng GitHub và remote/nhánh/lịch sử bằng Git. GitHub trả danh sách nhánh trống và `409 Git Repository is empty`; clone xác nhận không có commit, local HEAD là `main`. Đối chiếu đủ mã P0-01–P0-10/P1-01–P1-10 với nguồn, các ràng buộc, phụ thuộc/đầu ra/nghiệm thu và nội dung phù hợp công khai. Chạy 56 kiểm tra tĩnh bằng Node.js: đúng 5 file root, UTF-8, code fence/bảng Markdown, liên kết local, đủ 20 task với các cột bắt buộc, D-01–D-09/acceptance IDs và quét mẫu dữ liệu nhạy cảm; tất cả đạt. Đã rà soát staged diff/danh sách file; `git diff --cached --check` không báo lỗi. Chỉ kiểm chứng tài liệu/Git, **không chạy test ứng dụng, FFmpeg benchmark, Graph, LAN pilot hoặc UAT** vì chưa có triển khai và không thuộc yêu cầu lần này.
- **Việc còn mở / lý do:** chủ dự án review bộ tài liệu và Gate 1; hoàn thiện P0-01/P0-02 theo phạm vi giao sau; duyệt D-01–D-09 tại yêu cầu, đặc biệt overlap, sai lệch xưởng và mẫu video. IT chưa được xác minh về Graph, server, chính sách dữ liệu/retention; chưa có video benchmark được duyệt. Chờ yêu cầu riêng để bắt đầu lập trình P0-05/P0-06/P0-08; không tự chuyển sang code.

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

### 2026-11-19 — Kiểm tra lại FFmpeg/FFprobe theo chính sách máy

- **Yêu cầu / task:** xác minh môi trường thực tế cho P0-05/P0-06/P0-08; nếu thiếu, ghi tính năng còn thiếu và lệnh chạy lại trên máy có bộ công cụ được IT cho phép.
- **Trạng thái:** môi trường hiện tại **không đủ điều kiện end-to-end**. Không chạy pipeline upload/proxy/frame trong lượt này; không vượt Group Policy.
- **Lệnh đã chạy:** `Get-Command ffmpeg,ffprobe -All -ErrorAction SilentlyContinue`; `Get-AuthenticodeSignature -LiteralPath $ffmpegExe`; `& $ffmpegExe -version`; `& $ffmpegExe -hide_banner -encoders 2>&1 | Select-String -Pattern 'libx264|h264|png|mpeg4|msmpeg4'`; và lệnh tương tự với `-decoders`. `$ffmpegExe` là `C:\Program Files\SolidWorks Corp 2024\SOLIDWORKS\FloXpress\bin\ffmpeg.exe`.
- **Kết quả thực tế:** không có `ffmpeg` hoặc `ffprobe` trên PATH. FFmpeg vendor version 6.1, chữ ký Siemens hợp lệ; có decoder H.264, nhưng encoder list không có `libx264` hay encoder H.264 nào, và không có encoder PNG. Decoder list không có PNG. FFprobe ở `C:\Users\logtran\AppData\Local\Temp\motion-ffmpeg-tools\node_modules\@ffprobe-installer\win32-x64\ffprobe.exe` là file không ký số và đã bị Group Policy chặn trước đó; không chạy lại. Không có FFprobe đã xác minh/được phép trong PATH hoặc thư mục tiện ích đã kiểm tra.
- **Lệnh chạy lại trên máy được IT cho phép:** đặt `$env:FFMPEG` và `$env:FFPROBE` thành đường dẫn bộ cài được phép; xác nhận `Get-AuthenticodeSignature` theo chính sách IT, chạy `& $env:FFMPEG -version` và `& $env:FFPROBE -version`, rồi kiểm tra `& $env:FFMPEG -hide_banner -encoders 2>&1 | Select-String 'libx264|png'` và `& $env:FFMPEG -hide_banner -decoders 2>&1 | Select-String 'png'`. Chỉ chạy demo nếu có encoder `libx264`, encoder/decoder `png` và FFprobe hoạt động; khi đó làm theo các lệnh fixture, pytest, build và uvicorn trong `README_POC_LOCAL.md` tại workspace local.
- **Việc còn mở:** toàn luồng CFR/VFR/rotation, proxy-to-source, step sau seek, mở lại mốc, sai lệch/latency ứng dụng chưa chạy do thiếu công cụ. D-02, D-05 và go/no-go video xưởng vẫn mở. PR #1 vẫn chỉ được phép chứa 5 file Markdown; code/OpenAPI/test giữ local.

### 2026-11-19 — Tiếp tục POC local sau giới hạn Group Policy

- **Yêu cầu / task:** tiếp tục P0-05 + P0-06 + phần kiểm chứng kỹ thuật P0-08 tại workspace; áp dụng quy tắc GitHub chỉ nhận 5 file Markdown root và cập nhật PR #1.
- **Trạng thái:** có scaffold API FastAPI/UI React TypeScript và công cụ fixture local; **chưa đạt nghiệm thu end-to-end video POC**. PR #1 đang mở và Files changed chỉ gồm đúng 5 file được phép.
- **Thay đổi local:** upload MP4/MOV, trạng thái xử lý, frame selection theo ordinal hoặc proxy timestamp, yêu cầu ảnh source có frame token, Start/End và thời lượng source PTS, measurement JSON atomic, xác minh lại SHA/index khi mở; giới hạn upload và chặn vùng data đặt trong repo/OneDrive. Thêm hướng dẫn PowerShell, generator và checker fixture. Code, OpenAPI, cấu hình, test, fixture vẫn local.
- **Kiểm chứng thực tế:** `py -3.12 -m pytest -q tests/test_contract.py` — **5 passed**, một cảnh báo deprecated từ dependency; media/index và ảnh được stub trong các test API. `npm.cmd --prefix frontend run build` — thành công. Trước đó FastAPI/UI trả HTTP 200 và giao diện khởi động trong Chrome headless. Fixture nguồn CFR/VFR/rotation đã đối chiếu PTS/màu/chiều ảnh với manifest: sai lệch mốc Start, End, duration đều 0 µs ở các fixture giả lập; phép đo BMP nguồn median 73,9 ms, max 80,4 ms, không phải latency ứng dụng. `git diff --check` và kiểm tra Markdown/liên kết AC đạt.
- **Chưa xác minh:** không chạy executable FFprobe/Python venv bị Group Policy chặn theo xác nhận của người dùng. FFmpeg có sẵn thiếu `libx264` và PNG, nên chưa chạy tạo proxy H.264, so ánh xạ source/proxy end-to-end, ảnh exact-frame từ API, tương tác player với video, sai lệch/latency seek-step thực tế. Không có clip xưởng, không chốt D-02/D-05 và không đưa ra go/no-go.
- **Việc còn mở:** trên máy có bộ FFmpeg/FFprobe được IT cho phép, chạy demo với fixture CFR/VFR/rotation và đo Start/End/duration, mở lại, hai chiều step sau seek, đầu/cuối video và lỗi đầu vào; đo latency seek/step trên cấu hình ghi rõ. Sau đó chỉ kết luận P0-08 kỹ thuật theo bằng chứng fixture; D-02/D-05 và go/no-go xưởng vẫn cần quyết định riêng. SQLite nghiệp vụ, SharePoint, Excel, AI chưa triển khai.

### 2026-11-18 — P0-05/P0-06/P0-08 và giới hạn GitHub

- **Yêu cầu / task:** chủ dự án duyệt Gate 1, giao POC local và yêu cầu chỉnh PR #1 chỉ còn 5 file Markdown được phép trên GitHub; code, cấu hình, OpenAPI và test giữ local.
- **Trạng thái:** Gate 1 đã được duyệt cho phạm vi POC; PR #1 đang được chỉnh để review. Đã tạo code sơ bộ local cho API, index/timestamp, player, lưu mốc, test và fixture; chưa tuyên bố POC hoàn tất.
- **Thay đổi chính:** chuyển luồng, timestamp, ví dụ, API và AC-POC vào `ARCHITECTURE.md`; cập nhật yêu cầu/kế hoạch/quy tắc agent và giữ OpenAPI/code local. D-02/D-05 và go/no-go xưởng tiếp tục mở.
- **Kiểm chứng thực tế:** `npm.cmd --prefix frontend run build` thành công; `py -3.12 -m pytest -q tests/test_contract.py` đạt 4 test với **pipeline video và ảnh giả lập**, không phải test FFprobe/proxy thật. FastAPI trả HTTP 200 và giao diện khởi động trong Chrome headless; chưa thử tương tác video trong browser. Trên Windows 11 Enterprise, Python 3.12.9, Intel i5-1135G7, FFmpeg 6.1 có sẵn không có `libx264` hoặc PNG; FFprobe tải về và Python virtual environment bị Group Policy chặn. Người dùng xác nhận không cần chạy các executable bị chặn.
- **Fixture nguồn:** dùng FFmpeg có sẵn tạo 3 clip MPEG-4 tổng hợp CFR, VFR và rotation, 12 frame/clip; `tools/check_fixture_source.py` đối chiếu frame 3/8 với manifest và ảnh nguồn, sai lệch Start/End/duration đều 0 µs trên các mẫu này, rotation hiển thị đúng kích thước. Decode BMP nguồn 12 mẫu: median 73,9 ms, max 80,4 ms. Đây là phép đo decoder nguồn, **không phải** sai lệch hoặc latency seek/step của ứng dụng hay kết quả trên video xưởng.
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

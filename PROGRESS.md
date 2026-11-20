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

### 2026-11-20 — Rà soát tham chiếu Time Study có phát video

- **Yêu cầu / task:** sau khi làm rõ vị trí, xem phần Time Study/video trong một ứng dụng nội bộ khác để hiểu cách chọn mốc thời gian.
- **Kết quả đọc mã:** component trình duyệt dùng input file local, `URL.createObjectURL`, `<video>` native và `HTMLVideoElement.currentTime`; người dùng bấm timeline để seek và thêm breakpoint. Các breakpoint là số giây dạng float, tạo các khoảng liền kề từ đầu đến cuối clip; mốc cách nhau dưới 80 ms bị xem là trùng.
- **Lưu trữ và giới hạn:** backend nhận metadata/element start-end-duration và excluded periods; file video không được upload/lưu cùng phép đo, không có checksum hoặc source-frame/PTS reference. Không thấy frame-step, xác minh ảnh frame gốc hay mapping proxy/source. Cách này hữu ích làm UI thử nghiệm cho video browser hỗ trợ, nhưng chưa đủ cho kiểm chứng frame/PTS chính xác và VFR theo AC-POC-02/04.
- **Tác động tới Motion Tracking:** chưa đổi kiến trúc hoặc contract; có thể dùng tham khảo luồng chọn file, phát và timeline, nhưng vẫn cần chứng minh timestamp/frame identity theo Gate 1 trước khi xem xét thay media pipeline. Đây chỉ là rà soát mã nguồn tham chiếu; không sửa/chạy ứng dụng tham chiếu, không sửa code/test/clip Motion Tracking và không chạy test trong lượt này.
- **Việc còn mở:** PyAV chưa được xác nhận/cài/chạy theo chính sách IT; giải mã exact-frame, proxy H.264/PNG và E2E Motion Tracking chưa kiểm chứng. D-02, D-05 và quyết định go/no-go video xưởng vẫn mở.

### 2026-11-19 — Kiểm tra khả năng dùng PyAV thay FFmpeg CLI

- **Yêu cầu / task:** thử phương án PyAV local cho decode/PTS/time_base, proxy H.264, PNG và nếu đủ điều kiện thì tích hợp/test luồng POC.
- **Trạng thái:** chưa cài hoặc chạy PyAV. Chưa xác nhận được chính sách IT cho phép cài/chạy native extension mới; tuân thủ điều kiện của yêu cầu và không tải/cài package.
- **Lệnh kiểm tra đã chạy:** đặt `$env:PYTHONPATH = Join-Path (Get-Location) '.deps'`; chạy `py -3.12 -c "import importlib.util; s=importlib.util.find_spec('av'); print('av_spec='+str(s)); print('av_origin='+(s.origin if s else 'not-installed'))"`; kiểm tra thư mục `.deps` với `Get-ChildItem -LiteralPath .deps -Directory -Filter 'av*'`. Kết quả `av_spec=None`, `av_origin=not-installed`, không có distribution `av*` trong `.deps`.
- **Phạm vi chưa chạy:** không decode fixture CFR/VFR/rotation, không đọc/đối chiếu PTS/time_base hoặc hình; không kiểm tra encode proxy H.264/browser, trích PNG, tua/step, Start/End, lưu/mở lại hoặc latency. Vì chưa có kết quả giải mã/proxy, không tích hợp backend, không đổi kiến trúc/định dạng proxy và không kết luận phần nào của PyAV đã đạt.
- **Việc còn mở:** cần IT xác nhận việc cài và chạy PyAV wheel/native libraries được phép trên máy thử. Sau đó mới cài trong phạm vi local, ghi version/build/codec capabilities, chạy đối chiếu độc lập theo README và chỉ tích hợp nếu cả decode lẫn proxy/image đáp ứng contract. D-02, D-05 và quyết định go/no-go trên video xưởng vẫn mở; code, cấu hình, test và clip giữ local.

### 2026-11-19 — Thử chạy POC với bộ công cụ IT được phép

- **Yêu cầu / task:** chạy lại P0-05/P0-06 và phần kiểm chứng kỹ thuật P0-08 theo `README_POC_LOCAL.md` với FFmpeg/FFprobe được IT cho phép.
- **Trạng thái:** bị chặn trước khi chạy media. `FFMPEG`/`FFPROBE` không được cấu hình; `Get-Command ffmpeg,ffprobe -All -ErrorAction SilentlyContinue` không tìm thấy executable trên PATH. Chưa có đường dẫn bộ IT cho phép để thực hiện preflight trong runbook.
- **Đã kiểm tra:** đọc `AGENTS.md`, `README_POC_LOCAL.md`, `PROJECT_REQUIREMENT.md`, `ARCHITECTURE.md`, `IMPLEMENTATION_PLAN.md` và trạng thái/AC-POC trong file này. Xác nhận PR #1 vẫn mở và diff của PR chỉ gồm 5 file Markdown root được phép.
- **Không chạy / lý do:** không chạy FFmpeg/FFprobe hoặc test fixture/app vì không có bộ công cụ đã xác nhận IT cho phép. FFmpeg Siemens đã kiểm tra ở lần trước có chữ ký hợp lệ nhưng thiếu encoder H.264/`libx264`, encoder/decoder PNG và không có xác nhận IT cho phép; FFprobe trước đó bị Group Policy chặn. Không đổi Group Policy, không thử chạy lại executable bị chặn. Không có số đo sai lệch hay latency mới.
- **Việc còn mở:** cần đường dẫn tới FFmpeg/FFprobe được IT cho phép với H.264 encode và PNG encode/decode để chạy smoke test trong README, rồi chạy CFR/VFR/rotation qua upload → proxy → seek/step → Start/End → lưu/mở lại và đối chiếu ground truth. D-02, D-05 và quyết định go/no-go video xưởng vẫn mở. Code, cấu hình, test và video tiếp tục chỉ ở local.

### 2026-11-19 — Chuẩn bị lượt chạy lại POC trên máy được IT cho phép

- **Yêu cầu / task:** tiếp tục P0-05/P0-06 và phần kiểm chứng kỹ thuật P0-08; nếu chưa có máy/bộ công cụ phù hợp, chuẩn bị lệnh PowerShell chạy lại kiểm tra công cụ và bài test.
- **Trạng thái:** chưa có bộ FFmpeg/FFprobe vừa đủ tính năng vừa xác nhận được IT cho phép; **end-to-end CFR/VFR/rotation chưa chạy**. Không đổi Group Policy, không chạy lại FFprobe unsigned từng bị chặn.
- **Lệnh kiểm tra môi trường đã chạy:** `Get-Command ffmpeg,ffprobe -All -ErrorAction SilentlyContinue`; `Get-AuthenticodeSignature -LiteralPath $candidate`; `& $candidate -version`; `& $candidate -hide_banner -encoders 2>&1 | Select-String -Pattern 'libx264|h264|png|mpeg4|msmpeg4'`; lệnh tương tự với `-decoders`; `py -3.12 --version`; `node --version`; `npm.cmd --version`. `$candidate` là FFmpeg trong thư mục SolidWorks đã ghi ở mục nhật ký trước.
- **Kết quả công cụ:** FFmpeg/FFprobe không có trên PATH. FFmpeg 6.1 có Authenticode `Valid`, signer Siemens Industry Software Inc.; danh sách không có encoder `libx264`/H.264 hoặc PNG, decoder có H.264 nhưng không có PNG. Chữ ký không tự chứng minh IT phê duyệt. FFprobe hợp lệ/được IT cho phép chưa được xác minh. Python 3.12.9, Node v26.12.1 và `npm.cmd` 11.20.0 sẵn có; `.deps` và `frontend/node_modules` tồn tại. Gọi `npm --version` bị PowerShell chặn script `npm.ps1`, nên dùng `npm.cmd`, không đổi policy.
- **Kiểm tra local không cần media đã chạy:** `$env:PYTHONPATH = Join-Path (Get-Location) '.deps'; py -3.12 -m pytest -q tests/test_contract.py` → **5 passed, 1 DeprecationWarning**; các test thay pipeline media bằng stub. `npm.cmd --prefix frontend run build` → thành công. Đây không phải test video end-to-end.
- **Runbook local:** thêm `README_POC_LOCAL.md` với lệnh copy/paste để xác minh đường dẫn/chữ ký/capability, smoke test H.264 + encode/decode PNG + FFprobe, cài dependency, chạy contract test/build, tạo fixture/ground truth và khởi chạy app để kiểm tra thủ công upload → seek/step → Start/End → refresh/mở lại. Tài liệu runbook, code, cấu hình, test và video giữ local; GitHub chỉ nhận file trong danh sách 5 Markdown được phép.
- **Chưa chạy / việc còn mở:** thiếu FFprobe được IT cho phép, encoder `libx264`/H.264, encoder/decoder PNG. Chưa tạo proxy hoặc chạy app trên CFR/VFR/rotation; chưa so ảnh/PTS proxy-source, thao tác sau seek, Start/End/duration hoặc đo latency seek/step. Ghi cụ thể thiết bị và giá trị thực tế khi chạy được runbook. D-02, D-05 và quyết định go/no-go trên video xưởng vẫn mở.

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

### 2026-11-20 — Thiết kế lại workstation POC theo yêu cầu Motion Tracking

- **Yêu cầu / phạm vi:** dùng màn hình Time Study trong thư mục 15.CIOS làm tham khảo luồng thao tác, sau đó tạo giao diện phù hợp với POC cần phân biệt rõ playback proxy và frame gốc đã xác minh. Đây là thay đổi giao diện local, không mở rộng sang nghiệp vụ MVP.
- **Trạng thái:** đã cập nhật workstation React/TypeScript local; **chưa xác nhận video end-to-end** và chưa coi P0-05/P0-06/P0-08 là đạt nghiệm thu.
- **Thay đổi local:** nhóm màn hình thành nạp/xử lý video, thông tin codec/cadence/rotation/hash, playback proxy, ảnh frame gốc đã xác minh, điều khiển tiến/lùi theo ordinal nguồn, các thao tác Start/End, quay lại mốc và thời lượng theo timestamp nguồn. Nút đánh dấu tiếp tục bị khóa cho đến khi ảnh frame và token xác minh khớp. Các file giao diện và toàn bộ code/config/OpenAPI/test/fixture/video giữ ở workspace local.
- **Kiểm chứng thực tế:** npm.cmd --prefix frontend run build → exit code 0; script build hiện chạy tsc --project tsconfig.json, xác nhận TypeScript compile. git diff --check không báo lỗi. Đây không phải kiểm tra browser playback, API/video pipeline hay tương tác seek/step end-to-end.
- **Giới hạn và việc còn mở:** môi trường hiện chưa có FFprobe được IT cho phép và FFmpeg đầy đủ encoder H.264/PNG; theo yêu cầu trước đó không chạy executable bị Group Policy chặn, không đổi policy và không cài PyAV khi chưa được IT cho phép. Chưa đo sai lệch hoặc độ trễ ứng dụng trên clip CFR/VFR/rotation. D-02, D-05 và go/no-go trên video xưởng tiếp tục mở.
- **Bàn giao tài liệu:** thêm mục nhật ký này trong PR #1; GitHub chỉ nhận PROGRESS.md trong lần cập nhật này, còn code giao diện vẫn local. Chờ review giao diện và môi trường media được IT cho phép để chạy kiểm chứng thực tế.

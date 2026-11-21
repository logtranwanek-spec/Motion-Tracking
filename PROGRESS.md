# Tiến độ và nhật ký Motion Tracking / Video Time Study

Mốc cập nhật khởi tạo: **2026-11-18 (Asia/Bangkok)**. Nguồn tham khảo: tài liệu tổng hợp nội bộ v1.0 ngày 17/11/2026; không đưa file nguồn hoặc dữ liệu xưởng vào repo.

Liên kết: [yêu cầu](PROJECT_REQUIREMENT.md), [kiến trúc](ARCHITECTURE.md), [kế hoạch task](IMPLEMENTATION_PLAN.md), [quy tắc agent](AGENTS.md).

## Hiện trạng

| Hạng mục | Trạng thái thực tế hiện tại |
| --- | --- |
| Repository | Repo GitHub công khai; PR #1 đang mở trên `docs/gate-1-poc-contract` theo `main`, Files changed hiện đúng 5 file Markdown root. Code, OpenAPI, cấu hình và test ở local. |
| Bộ tài liệu | Chỉnh PR #1 để chỉ thay đổi 5 file root; nội dung Gate 1 được chuyển vào các file này. OpenAPI chi tiết và code ở workspace local. |
| P0-01 | Chủ dự án xác nhận duyệt Gate 1 cho POC; các quyết định nghiệp vụ khác vẫn mở. |
| P0-02 | Contract POC ban đầu được duyệt trong Gate 1; yêu cầu mới về video user-local làm contract media cũ không còn phù hợp. Contract API local-only cần cập nhật sau khi chốt D-10. |
| P0-03 | Đã có skeleton frontend/backend, `run.cmd`, README và test ở local; chưa hoàn tất setup/kiểm thử trên máy theo policy IT hoặc CI lint/test. |
| D-10 / policy runtime | Chủ dự án chọn browser-native local preview làm thử nghiệm đầu vì không cần tải/cài media engine. Frontend đã bỏ POST video; raw PTS, VFR exact-frame, proxy, lưu phép đo và network evidence chưa xác minh. D-10 vẫn mở. |
| P0-04 đến P0-08 | Browser feasibility slice cho P0-05/P0-06 đã viết local: mở/phát file browser Blob URL, scrub theo giây, hiển thị frame callback telemetry; không tạo proxy/index và khóa frame-step/Start/End. Backend còn legacy upload endpoints, không expose. Chưa chạy app/video hoặc benchmark. Chưa có SQLite schema/migration hoặc P0-07. |
| P0-09 | Chưa xác minh với IT, chưa thử Graph; SharePoint còn có điều kiện, local-only là hướng lõi. |
| P0-10 | Mới có nguyên tắc sync/recovery trong kiến trúc, chưa thiết kế chi tiết được nghiệm thu hoặc triển khai recovery/sync. |
| Phase 0 POC | **Browser-only preview đã viết local nhưng chưa chạy/xác minh. Exact PTS, frame-step, Start/End, persistence và proxy chưa làm; backend legacy upload còn tồn tại. Không dùng video xưởng; chưa benchmark hoặc go/no-go.** |
| Phase 1 MVP | **Chưa bắt đầu lập trình, chưa test ứng dụng, chưa pilot LAN hoặc UAT.** |
| Phase 2–4 | AI, SOP, tích hợp và phân tích nâng cao chỉ là lộ trình đề xuất, chưa được triển khai. |

Không dùng trạng thái hoàn tất của lần soạn tài liệu để suy ra POC/MVP đã hoàn tất. Ngày/SHA và kết quả kiểm tra GitHub sau push được báo trong bàn giao; nội dung nhật ký không ghi trước một kết quả push chưa xảy ra.

## Quy tắc nhật ký

Sau mỗi yêu cầu đã thực hiện, thêm một mục trong cùng commit hoặc PR với thay đổi, kể cả làm một phần hoặc bị vướng. Giữ các mục cũ. Mỗi mục có ngày, yêu cầu/mã task, trạng thái, thay đổi chính, kết quả kiểm chứng và việc còn mở. Cập nhật bảng hiện trạng khi có tiến triển; lịch sử cũ không bị xóa. Khi đổi nghiệp vụ/thiết kế/kế hoạch/quy tắc agent, cập nhật mọi file liên quan theo [AGENTS.md](AGENTS.md).

## Nhật ký

### 2026-11-20 — Kiểm tra engine có sẵn trên Windows

- **Yêu cầu / phạm vi:** xem máy Windows có engine media tích hợp tương tự khả năng mở video của Clipchamp không, để cân nhắc thay FFprobe/FFmpeg CLI.
- **Kết quả nghiên cứu:** Windows Media Foundation có Source Reader để đọc media cục bộ và trả timestamp sample theo đơn vị 100 ns; Microsoft cảnh báo seek không phải lúc nào cũng frame-accurate. Clipchamp hỗ trợ nhiều container như MP4/MOV nhưng có thể tự chuyển đổi khi import; đây là ứng dụng biên tập, chưa xác minh là API/engine dùng được trong Motion Tracking.
- **Thay đổi tài liệu:** thêm Media Foundation vào danh sách ứng viên D-10 trong yêu cầu và kiến trúc, kèm giới hạn helper native, timestamp và seek. Chưa chuyển ứng viên thành quyết định.
- **Kiểm chứng:** đọc tài liệu Microsoft Learn/Support chính thức; không kiểm tra phiên bản/capability trên máy, không viết/chạy helper, không cài engine hoặc đọc video.
- **Việc còn mở:** cần xác minh codec MP4/MOV, HEVC nếu có, VFR/PTS, rotation, frame-step/seek và proxy H.264/PNG trên workstation được IT cho phép. D-10 chưa chốt; D-02, D-05 và go/no-go video xưởng vẫn mở.

### 2026-11-20 — Đánh giá lựa chọn khác ngoài FFprobe

- **Yêu cầu / phạm vi:** xác định có thể dùng runtime khác ngoài FFprobe cho Motion Tracking không.
- **Kết quả nghiên cứu:** PyAV cung cấp truy cập frame PTS/time_base nhưng là binding của FFmpeg libraries; có thể thay CLI FFprobe chứ không loại bỏ nền tảng FFmpeg. GStreamer có API discovery và PTS trên buffer, nhưng plugin decode/encode phải được kiểm tra theo máy. Browser-native `<video>` phù hợp mở/phát local như 15.CIOS, nhưng chưa được xác minh cho source-frame PTS/VFR theo AC-POC.
- **Thay đổi tài liệu:** bổ sung các ứng viên và giới hạn vào D-10 trong `PROJECT_REQUIREMENT.md` và `ARCHITECTURE.md`; không chọn hoặc cài runtime.
- **Kiểm chứng:** đối chiếu tài liệu chính thức PyAV/GStreamer; không cài/chạy package, không chạy media tools, không mở video hoặc đổi Group Policy.
- **Việc còn mở:** cần IT xác nhận runtime/plugin được phép rồi thử trên fixture CFR/VFR/rotation, so frame/PTS, proxy H.264/PNG và chứng minh file không gửi lên server. D-10 chưa chốt; D-02, D-05 và go/no-go video xưởng vẫn mở.

### 2026-11-20 — Chẩn đoán POC không mở được video

- **Yêu cầu / task:** kiểm tra lỗi mở video hiện tại, liên quan P0-05/P0-06.
- **Kiểm tra ứng dụng:** port `127.0.0.1:8000` đang có FastAPI chạy. Các GET tới `/`, stylesheet, React, ReactDOM và bundle `/static/dist/app.js` đều trả HTTP 200; frontend/server khởi động và phục vụ asset được.
- **Kết quả thực tế:** metadata xử lý gần nhất trong vùng dữ liệu local báo `status=FAILED`, `error.code=TOOL_FAILED`, Windows `WinError 2` (không tìm thấy executable). Không có `ffprobe`/`ffmpeg` trên PATH hiện tại và không có biến `FFPROBE`/`FFMPEG` trong shell hiện tại. Theo mã `backend/video.py`, bước đầu tiên gọi FFprobe để đọc PTS; vì vậy processing dừng trước khi tạo proxy, player không có video để phát. Sau bước đó, máy còn cần FFmpeg với encoder `libx264` và PNG theo runbook.
- **Phạm vi đã kiểm tra:** đọc mã frontend/backend và `run.cmd`; chỉ đọc metadata trạng thái/lỗi, không mở/đọc nội dung video. Không chạy FFmpeg/FFprobe, không đổi Group Policy, không upload thêm video và không chạy test media.
- **Việc còn mở:** để luồng cũ xử lý được cần bộ FFprobe/FFmpeg được IT cho phép, cấu hình đúng `FFPROBE`/`FFMPEG` và đủ capability. Tuy nhiên POC vẫn upload file đến FastAPI host, không đáp ứng yêu cầu user-local khi server nằm trên LAN; cần thay bằng media processor trên workstation trước khi dùng video thật. D-10 còn mở cho runtime; D-02, D-05 và go/no-go xưởng vẫn mở.

### 2026-11-20 — Yêu cầu lưu video trên workstation của user

- **Yêu cầu / phạm vi:** user xác nhận video phải nằm trên máy của user, không lưu trên server; dùng luồng chọn file local của 15.CIOS làm tham khảo.
- **Thay đổi chính:** cập nhật yêu cầu, kiến trúc và kế hoạch để original không upload lên API/server; proxy/index/frame cache nếu có cũng xử lý/lưu local theo phương án còn cần chốt. Bổ sung nguyên tắc cho agent. D-10 chỉ còn mở về runtime local, quyền truy cập/mở lại file, artifact/cache và contract metadata-only.
- **Kiểm tra thực tế:** đọc `frontend/src/app.tsx` và `backend/app.py`; hiện UI gửi `multipart/form-data` tới `POST /api/poc/videos`, FastAPI nhận file và lưu tại vùng dữ liệu của host backend. Do đó khi backend chạy trên server LAN, video sẽ ở server; khi chạy `run.cmd` trên cùng PC, host backend chính là PC đó. Đọc `README_POC_LOCAL.md`; runbook hiện mô tả luồng upload này.
- **Kết quả kiểm chứng:** kiểm tra tĩnh bằng tìm kiếm mã nguồn; không chạy app, media tools hoặc test trong lượt này. Chưa thay code; demo hiện tại chưa đáp ứng yêu cầu không upload. Đã thêm cảnh báo vào runbook local để không dùng video nhạy cảm/xưởng với luồng cũ.
- **Việc còn mở:** chốt/đánh giá D-10 và thay media API/OpenAPI, worker và player theo hướng workstation-local; chứng minh bằng Network capture rằng không gửi video bytes/path lên server; kiểm thử PTS, VFR, rotation, exact-frame, lưu/mở lại. D-02, D-05 và go/no-go video xưởng vẫn mở.

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

### 2026-11-20 — Thêm launcher local cho POC

- **Yêu cầu / phạm vi:** tạo file run.cmd để khởi chạy app local từ thư mục gốc. Launcher chỉ phục vụ workspace POC và không đưa lên GitHub.
- **Thay đổi local:** run.cmd kiểm tra Python 3.12 và dependency hiện có, đặt PYTHONPATH tới .deps, build frontend bằng npm.cmd, đặt vùng dữ liệu mặc định trong %LOCALAPPDATA%\MotionTrackingPOC nếu người dùng chưa cấu hình, rồi chạy Uvicorn tại 127.0.0.1:8000. Cập nhật README_POC_LOCAL.md cách khởi chạy và điều kiện media. Launcher không tự cài dependency, không gọi/tải FFmpeg hoặc FFprobe, không mở cổng LAN và không thay đổi Group Policy.
- **Kiểm chứng thực tế:** rà soát tĩnh lệnh trong run.cmd theo hướng dẫn chạy hiện có; chưa chạy launcher hoặc server trong lượt này. Không có kiểm tra phát/seek video mới. Ghi nhận run.cmd và README_POC_LOCAL.md ở local; PR #1 chỉ nhận mục nhật ký này trong PROGRESS.md.
- **Việc còn mở:** cần chạy run.cmd trên máy để xác nhận server khởi động; xử lý video vẫn cần FFmpeg/FFprobe được IT cho phép và cấu hình trong FFMPEG/FFPROBE. D-02, D-05 và go/no-go video xưởng tiếp tục mở.

### 2026-11-20 — Chỉnh giao diện workstation theo hướng chuyên nghiệp

- **Yêu cầu / phạm vi:** làm giao diện Motion Tracking gọn và chuyên nghiệp hơn, giữ nguyên quy trình nghiệp vụ và contract POC đã duyệt.
- **Thay đổi local:** làm rõ luồng 3 bước Video nguồn → Xác minh frame → Time study; chuẩn hóa typography, màu sắc, khoảng cách, panel và trạng thái; tăng phân biệt trạng thái local/xử lý/xác minh; thêm vị trí frame cạnh điều khiển; làm rõ hành động Start/End; cải thiện bố cục responsive, focus bàn phím và tôn trọng reduced-motion. Cập nhật title trình duyệt. Chỉ sửa giao diện local, không đổi API hay timestamp.
- **Kiểm chứng thực tế:** npm.cmd --prefix frontend run build hoàn tất với exit code 0; script build chạy tsc --project tsconfig.json. Chưa chạy browser để xem giao diện, chưa thao tác video và chưa chạy media end-to-end.
- **Việc còn mở:** review giao diện trên browser và xác minh tương tác thực khi có công cụ FFmpeg/FFprobe được IT cho phép. D-02, D-05 và quyết định go/no-go video xưởng tiếp tục mở. Mã giao diện giữ local; PR #1 chỉ nhận nhật ký PROGRESS.md.

### 2026-11-20 — Làm lại giao diện theo phong cách 24.IAH&DP

- **Yêu cầu / phạm vi:** người dùng yêu cầu giao diện Motion Tracking theo phong cách dự án 24.IAH&DP sau khi chưa hài lòng với bản polish trước. Đã xem Dashboard CSS và cấu trúc workspace/sidebar/topbar của dự án tham khảo; chỉ dùng làm tham khảo ngôn ngữ thiết kế, không lấy nghiệp vụ hoặc code chức năng của dự án đó.
- **Thay đổi local:** chuyển POC sang shell nền tối với sidebar workflow, workspace card, breadcrumb/topbar, panel tối viền mảnh và xanh dương làm màu nhấn. Điều hướng sidebar đưa tới Video source, Frame review và Time study; badge thể hiện Ready/Verified/Saved. Tinh chỉnh responsive để sidebar thành navigation gọn trên màn hình nhỏ. Giữ luồng upload, proxy, xác minh ảnh frame gốc, PTS, step và Start/End.
- **Kiểm chứng thực tế:** npm.cmd --prefix frontend run build hoàn tất với exit code 0; script build chạy tsc --project tsconfig.json. Chưa chạy browser để xem render hoặc thao tác video; không chạy media end-to-end.
- **Việc còn mở:** cần xem giao diện trên browser để đánh giá trực quan và tiếp tục chỉnh theo phản hồi. D-02, D-05 và go/no-go video xưởng tiếp tục mở. Code/CSS giữ local; PR #1 chỉ nhận mục nhật ký này trong PROGRESS.md.

### 2026-11-21 — Đánh giá phương án media theo policy công ty

- **Yêu cầu / phạm vi:** tìm các cách khả dụng khi user không thể tải phần mềm từ Internet hoặc chạy file `.exe`. Không tìm cách vượt Group Policy, không cài/chạy công cụ media.
- **Trạng thái:** nghiên cứu giải pháp hoàn tất ở mức tài liệu; D-10 chưa được quyết định và luồng POC chưa được kiểm thử bằng browser/media trên máy user.
- **Thay đổi chính:** ghi ràng buộc runtime mới vào `AGENTS.md`, `PROJECT_REQUIREMENT.md`, `ARCHITECTURE.md`, `IMPLEMENTATION_PLAN.md`. Ưu tiên đề xuất là ứng dụng được IT host trong LAN, browser mở/phát file local bằng API sẵn có; giữ phép đo khóa đến khi source frame/PTS được xác minh. Ghi WebCodecs + demuxer JavaScript là hướng cần IT/security duyệt; Media Foundation/PyAV/FFmpeg/GStreamer chỉ khả thi nếu IT triển khai. Clipchamp/Photos/Media Player chỉ dùng kiểm tra playback thủ công, không thay media API của app.
- **Kiểm chứng thực tế:** đọc tài liệu chính thức Microsoft về Source Reader, codec và seek; W3C WebCodecs/WICG frame callback; Chrome WebCodecs. Tài liệu xác nhận Source Reader có timestamp nhưng seek có thể tới keyframe trước và cần đọc tiến; WebCodecs không bao gồm demux container; `requestVideoFrameCallback` báo thời gian trên browser media timeline. `git diff --check` không báo lỗi. Không chạy app, không mở video, không kiểm tra codec/PTS, không cài thư viện hay chạy executable.
- **Việc còn mở:** IT cần xác nhận phương thức host web app nội bộ và quy trình duyệt JavaScript bundle/dependency nếu chọn WebCodecs; cần benchmark khả năng frame/PTS, VFR, rotation, H.264/PNG và reopen trên browser/cấu hình user được phép. `run.cmd` hiện dựa vào Python/Node launcher có sẵn và chưa xác minh được policy cho phép; không coi đây là cách chạy đã được duyệt. D-02, D-05 và go/no-go video xưởng tiếp tục mở.

### 2026-11-21 — Thực hành browser-only local preview

- **Yêu cầu / phạm vi:** chọn và thử phương án media phù hợp nhất với policy không tải phần mềm/không chạy `.exe`; giữ video trên workstation và ưu tiên đường chạy trong browser có sẵn.
- **Trạng thái:** đã viết feasibility preview local; chưa chạy bằng browser hoặc video thật. P0-05/P0-06/P0-08 chưa nghiệm thu.
- **Thay đổi chính:** tạo `BROWSER_PREVIEW_LOCAL.html` dạng một file HTML/JavaScript/CSS độc lập để mở trong Edge/Chrome hiện có, chọn file qua picker, phát MP4/MOV bằng browser decoder, scrub và tua ±5 giây, hiển thị `requestVideoFrameCallback` telemetry. Trang không có thư viện ngoài/API upload. Cập nhật React source/CSS local theo cùng hướng, nhưng chưa build. Viết lại `README_POC_LOCAL.md` với cách mở trang và giới hạn. Cập nhật D-10, kiến trúc, plan và hướng dẫn agent. Các nút frame-step/Start/End giữ khóa; không tính duration nghiệp vụ. Backend còn legacy upload endpoints, không expose lên LAN.
- **Kiểm chứng thực tế:** rà soát source và README; tìm trong standalone HTML không thấy `fetch`, `FormData`, XHR, URL ngoài, script ngoài hoặc stylesheet ngoài. Đây chỉ là kiểm tra tĩnh; chưa mở HTML trong browser, chưa chọn/giải mã video, chưa kiểm tra Edge/Chrome codec, rotation, VFR, timestamp hoặc network capture. Không cài/chạy executable, không chạy `run.cmd`, npm/Python/FFmpeg.
- **Việc còn mở:** cần browser thử với fixture CFR/VFR/rotation và network capture trên máy được phép. Để đạt mục tiêu exact-frame, cần source PTS/index/demuxer được xác minh và persistence manifest local; nếu browser-only không đủ thì IT phải duyệt WebCodecs + demuxer hoặc triển khai local processor. Backend upload endpoint cần được loại/khóa trước khi host frontend/API cho user. D-10, D-02, D-05 và go/no-go video xưởng vẫn mở.

### 2026-11-21 — Thực hành browser-only local preview

- **Yêu cầu / phạm vi:** chọn và thử phương án media phù hợp nhất với policy không tải phần mềm/không chạy `.exe`; giữ video trên workstation và ưu tiên đường end-to-end trong browser.
- **Trạng thái:** đã viết feasibility preview local; chưa chạy bằng browser hoặc video thật. P0-05/P0-06/P0-08 chưa nghiệm thu.
- **Thay đổi chính:** tạo `BROWSER_PREVIEW_LOCAL.html` dạng một file HTML/JavaScript/CSS độc lập để mở trong Edge/Chrome hiện có, chọn file qua picker, phát MP4/MOV bằng browser decoder, scrub và tua ±5 giây, hiển thị `requestVideoFrameCallback` telemetry. Trang không có thư viện ngoài/API upload. Cập nhật React source/CSS local theo cùng hướng, nhưng chưa build. Viết lại `README_POC_LOCAL.md` với cách mở trang và giới hạn. Cập nhật D-10, kiến trúc, plan và hướng dẫn agent. Các nút frame-step/Start/End giữ khóa; không tính duration nghiệp vụ. Backend còn legacy upload endpoints, không expose lên LAN.
- **Kiểm chứng thực tế:** chỉ rà soát tĩnh source và nội dung README; chưa mở HTML trong browser, chưa chọn/giải mã video, chưa kiểm tra Edge/Chrome codec, rotation, VFR, timestamp hoặc network capture. Không cài/chạy executable, không chạy `run.cmd`, npm/Python/FFmpeg.
- **Việc còn mở:** cần browser thử với fixture CFR/VFR/rotation và network capture trên máy được phép. Để đạt objective đo exact frame, cần source PTS/index/demuxer được xác minh và persistence manifest local; nếu browser-only không đủ thì IT phải duyệt WebCodecs + demuxer hoặc triển khai local processor. Backend upload endpoint cần được loại/khóa trước khi host frontend/API cho user. D-10, D-02, D-05 và go/no-go video xưởng vẫn mở.

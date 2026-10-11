# Hướng dẫn sử dụng V-Reup Translator

V-Reup Translator là ứng dụng Windows giúp trích xuất phụ đề từ video, dịch nội dung, tạo giọng đọc và xuất video có phụ đề/lồng tiếng. Hướng dẫn này dùng tên điều khiển của giao diện **Tiếng Việt**.

## 1. Bắt đầu

### Mở ứng dụng

Với bản portable, giải nén toàn bộ gói vào thư mục có quyền ghi rồi mở `VReupTranslator.exe`. Với bản EXE một tệp, đặt EXE trong thư mục có quyền ghi và mở ứng dụng. Giữ các tệp đi kèm của bản portable trong cùng gói.

Ứng dụng tạo `settings.json` và thư mục `output` cạnh EXE; khi chạy từ mã nguồn, chúng nằm tại thư mục chứa `main.py`. Có thể chọn thư mục đầu ra khác trong giao diện.

1. Mở **Cấu hình → License**, kiểm tra trạng thái Free/Paid và quota hoặc hạn sử dụng. Nếu có License Key, nhập và xác minh theo hướng dẫn trên giao diện.
2. Kết nối mạng để xác minh quyền sử dụng khi mở ứng dụng. Free cần kết nối để xác minh quota; Paid cần xác minh ở mỗi lần mở, sau đó có thể dùng offline trong phiên đã xác minh đến hạn license. Dịch và giọng đọc trực tuyến vẫn cần mạng.
3. Chọn **Ngôn ngữ giao diện → Tiếng Việt** trong **Cấu hình** nếu cần.
4. Dùng một video ngắn để thử quy trình trước khi xử lý cả danh sách.

### Những gì cần chuẩn bị

| Công việc | Đầu vào cần có |
| --- | --- |
| OCR | Video có chữ/phụ đề nhìn thấy trên hình; model OCR sẵn sàng |
| Nhận diện giọng nói (ASR) | Video có âm thanh lời thoại; model nhận diện, tải lần đầu nếu chưa có |
| Dịch | Phụ đề và API key/model Gemini hoặc OpenAI hợp lệ |
| Lồng tiếng | Phụ đề đã kiểm tra và giọng đọc; credentials hoặc model tùy nhà cung cấp |
| Xuất video | Video nguồn, phụ đề/audio muốn dùng, FFmpeg và các ảnh logo đã chọn |

OCR và ASR nhận diện nội dung nguồn; cần chạy bước **Dịch** để có bản dịch. Nếu đã có SRT/ASS, có thể nhập trực tiếp và bỏ qua nhận diện.

## 2. Làm quen giao diện

Menu trái có các mục **Trích xuất phụ đề**, **Dịch thuật**, **Lồng tiếng**, **Cài đặt API**, **Cấu hình** và **Liên hệ**, cùng công cụ mở/lưu dự án. Phần làm việc gồm preview video, các tùy chọn của mục đang mở và nhật ký/tiến trình.

- Bốn nút xử lý chung là **Trích xuất Phụ đề**, **Dịch phụ đề**, **Lồng tiếng**, **Xuất video**. Mỗi nút chạy riêng bước tương ứng khi đủ dữ liệu.
- Preview cho phép chọn video, phát/tạm dừng, tua, zoom và xem toàn màn hình; nhấn **Esc** để thoát toàn màn hình.
- Checkbox trên các thẻ video chọn tập video cần xử lý. Khi có video được tick, các bước dùng tập đó; nếu không tick, ứng dụng dùng video đang chọn. Khi chưa xác định được video mục tiêu, ứng dụng có thể hỏi xử lý toàn bộ danh sách.
- Khối **Xuất phụ đề · JSON → SRT / ASS / VTT** chứa thiết bị, độ phân giải, **Dừng sau**, checkpoint và **Chạy pipeline / Tạm dừng / Dừng**.
- Nút bước xử lý màu xanh biểu thị hoàn tất, vàng là đang chạy, đỏ là có lỗi. Rê chuột lên nút để xem trạng thái chi tiết.
- Kéo vạch chia để chỉnh độ rộng cột. Nếu nhật ký bị ẩn khi cửa sổ hẹp, dùng nút **Nhật ký** để mở lại.

## 3. Thực hành: từ video đến video lồng tiếng

Ví dụ: bạn có video với phụ đề tiếng Trung trên hình và muốn xuất video có phụ đề, giọng đọc tiếng Việt. Hãy chạy từng bước để kiểm tra kết quả trước khi chuyển bước tiếp theo.

### Bước 1 — Thêm video và chọn nơi lưu

1. Mở **Trích xuất phụ đề**, tại **Nguồn video** chọn **Một video**.
2. Dùng nút chọn tại **Video / thư mục** để mở video. Các định dạng đầu vào gồm MP4, MKV, MOV, AVI, WEBM và M4V.
3. Chọn thư mục **Đầu ra** có quyền ghi và đủ dung lượng cho phụ đề, audio, video xuất.
4. Phát thử và chọn video trên preview để xác nhận đúng nguồn.
5. Bấm **Lưu Dự Án** và chọn tên tệp JSON để giữ dữ liệu làm việc qua lần mở ứng dụng sau.

**Kết quả cần thấy:** video xuất hiện trong preview, có thể phát/tua và thư mục đầu ra đã được chọn.

### Bước 2 — Trích xuất phụ đề

Trong **Nhận diện & vùng chọn**, chọn phương pháp phù hợp:

| Phương pháp | Khi nào dùng |
| --- | --- |
| **OCR Local** | Đọc chữ/phụ đề đã có trên hình |
| **Nhận diện giọng nói (ASR)** | Nhận diện lời thoại từ âm thanh |
| **ASR + OCR (đối chiếu)** | Đối chiếu lời thoại với phụ đề tiếng Trung trên hình, tìm câu thiếu/khác |

Với **OCR Local**:

1. Tua tới khung hình có phụ đề. Bấm **Chọn vùng thoại / TTS**, di chuyển và kéo các cạnh/góc để vùng bao trọn phụ đề.
2. Giữ **Dùng vùng thoại làm vùng quét OCR** nếu muốn quét cùng vùng. Bỏ chọn tùy chọn này và dùng nút chọn vùng OCR khi cần vùng quét riêng. Vùng TTS cũng dùng cho vị trí phụ đề; đây không phải nguồn âm thanh của ASR.
3. Chọn chế độ OCR nhanh hoặc chính xác. Có thể giữ **Mẫu dò / giây** mặc định cho lần thử đầu.
4. Nhập mốc bắt đầu/kết thúc đoạn nhận diện; để trống kết thúc để chạy tới hết video. Đây là phạm vi nhận diện, khác với các đoạn cắt lúc xuất video.

Với **ASR**, chọn model (mặc định `small`), đặt **Ngôn ngữ nguồn** trong **Cấu hình** và giữ VAD nếu muốn bỏ qua khoảng không có lời thoại khi nhận diện. Model chưa có cần tải lần đầu. Căn chỉnh WhisperX là tùy chọn; xem [hướng dẫn ASR và đối chiếu](speech-recognition.md) để biết runtime, GPU và ngôn ngữ hỗ trợ.

Chọn **CPU** trong khối xuất cho lần thử đầu; nếu dùng GPU, bấm **Kiểm tra** trước. Sau đó bấm **Trích xuất Phụ đề** và theo dõi nhật ký.

**Kết quả cần thấy:** phụ đề được tạo, có thể xem trong **Dịch thuật**; đầu ra có JSON và các bản SRT/ASS/VTT. Với chế độ kết hợp, dùng **Đối chiếu ASR + OCR** để xem câu cần kiểm tra và chọn nội dung phù hợp.

### Bước 3 — Cấu hình dịch

1. Mở **Cài đặt API**, chọn nhà cung cấp **Gemini** hoặc **OpenAI (ChatGPT)**.
2. Thêm/chọn profile, nhập API key cho tài khoản/project đang chọn. Project ở đây là nhãn quản lý tài khoản cục bộ.
3. Chọn model; dùng **Tải model** nếu cần lấy danh sách. Với OpenAI, kiểm tra URL API phù hợp với dịch vụ bạn sử dụng.
4. Bấm **Lưu tất cả cấu hình**, sau đó dùng chức năng test model/profile để kiểm tra kết nối. Test gửi yêu cầu thật và có thể dùng quota hoặc phát sinh phí của nhà cung cấp.
5. Trong **Cấu hình**, đặt **Ngôn ngữ nguồn** phù hợp hoặc **Tự nhận diện**, **Ngôn ngữ đích → Việt** và giữ **Thinking → Mặc định** nếu chưa có nhu cầu khác.

**Dịch song song nhiều profile** mặc định bật. Chỉ profile có key/model hợp lệ tham gia; phạm vi nhà cung cấp còn phụ thuộc cấu hình API. Nếu tắt tùy chọn này, mỗi tệp phụ đề được gửi trong một yêu cầu toàn bộ; tệp quá lớn có thể vượt giới hạn dịch vụ. Với lần thử đầu, chỉ cần một profile hợp lệ.

API key được lưu trong `settings.json` của ứng dụng, không nằm trong file dự án. Không chia sẻ settings khi chưa loại bỏ khóa; file xuất cấu hình API không chứa key và máy khác cần nhập key lại.

### Bước 4 — Dịch và kiểm tra phụ đề

1. Chọn đúng video trên preview rồi bấm **Dịch phụ đề**.
2. Mở **Dịch thuật** để kiểm tra bảng phụ đề. Cột **Gốc** là nguồn dịch; cột **Đã dịch** dùng cho preview, xuất phụ đề và lồng tiếng.
3. Sửa tên riêng, câu sai OCR và câu dịch chưa tự nhiên trực tiếp trong bảng. Chọn các dòng cần làm lại và dùng **Dịch đã chọn** nếu chỉ muốn dịch lại phần đó.
4. Phát/tua tới các câu đã sửa để kiểm tra thời gian. Có thể gộp, xóa, sửa timestamp hoặc hoàn tác thao tác trong editor.
5. Dùng **Lưu SRT** để lưu phụ đề hoặc **Xuất SRT mới** để tạo bản riêng. Đồng thời giữ dự án đã lưu để lưu cả dữ liệu editor và liên kết theo video.

Kết quả dịch được lưu sau từng batch thành công; batch sau lỗi hoặc bị hủy vẫn giữ phần đã lưu. Khi dịch ghi vào nguồn phụ đề đang mở, ứng dụng tạo bản sao lưu `*.backup-*` trước lần ghi đầu tiên của lượt dịch.

**Kết quả cần thấy:** các câu cần dùng có bản dịch đúng ngôn ngữ, thời gian phù hợp và hiển thị đúng khi xem video.

### Bước 5 — Tạo giọng đọc

1. Mở **Lồng tiếng → Giọng đọc**, chọn nhà cung cấp.
2. Với **Edge**, chọn ngôn ngữ và giọng; có thể dùng `vi-VN-HoaiMyNeural` hoặc `vi-VN-NamMinhNeural`. Edge cần mạng nhưng không cần API key Gemini/Vbee/NTSGenAI.
3. Với **VieNeu-TTS (Local)**, dùng nút tải model và chờ trạng thái sẵn sàng. Chọn **Tự động** hoặc **CPU**; nếu muốn NVIDIA GPU, tải bộ chạy/model GPU bằng nút riêng và kiểm tra thiết bị trước. Sau khi tải đủ, VieNeu có thể tạo tiếng offline.
4. Với dịch vụ khác, cấu hình tài khoản tương ứng trong **Cài đặt API** và kiểm tra giọng được hỗ trợ. Dùng **Tải giọng** khi nhà cung cấp có hỗ trợ.
5. Chọn giọng, Rate/Pitch/Volume và định dạng WAV hoặc MP3. Nên giữ giá trị mặc định khi thử lần đầu.
6. Chọn video cần lồng tiếng rồi bấm nút **Lồng tiếng** trên hàng xử lý chung. Nếu chạy trực tiếp trong tab Lồng tiếng, kiểm tra tệp đầu vào/nơi xuất và dùng **Dùng SRT đang chỉnh** để lấy nội dung editor hiện tại.

Chọn video khác trên preview sẽ đồng bộ phụ đề đầu vào tương ứng; hãy kiểm tra lại trước khi chạy. Với bản nháp chưa lưu, thao tác dùng SRT đang chỉnh tạo bản nháp mới để đọc đúng nội dung hiện tại.

**Kết quả cần thấy:** audio được tạo và nhật ký báo đường dẫn. Nghe thử trước khi xuất video.

Nếu có dòng đỏ do lời đọc dài hơn khoảng thời gian, đó là cảnh báo cần kiểm tra: ứng dụng vẫn đọc đủ câu và có thể chồng tiếng với câu sau. Có thể rút gọn nội dung, chỉnh thời gian hoặc chủ động đổi Rate rồi tạo tiếng lại. Timeline điều chỉnh nằm trong các tệp `.timed.*`; tệp đầu vào được giữ nguyên. Audio có thể giữ lời vượt cuối video, nhưng video xuất sẽ cắt phần vượt thời lượng video.

### Bước 6 — Xuất video

1. Trong **Trích xuất phụ đề**, chỉnh **Cắt & trộn âm thanh** nếu cần: đoạn cắt, âm lượng tiếng gốc và tiếng lồng.
2. Trong **Phụ đề**, đặt kiểu chữ, màu, vị trí và các tùy chọn hiển thị. Trong **Logo Text**, thêm logo/text/blur nếu cần; kiểm tra các ảnh logo vẫn tồn tại.
3. Xem preview để kiểm tra phụ đề và hiệu ứng. Độ phân giải và đoạn cắt áp dụng khi xuất; timestamp nhận diện vẫn theo video nguồn.
4. Chọn độ phân giải, thiết bị xuất rồi bấm **Xuất video**. Nếu thiếu phụ đề/audio, đọc thông báo và chỉ tiếp tục khi đầu ra đó đúng với nhu cầu.
5. Chờ nhật ký báo hoàn tất, mở video đầu ra và kiểm tra hình, phụ đề, tiếng gốc/tiếng lồng cùng đoạn đầu/cuối.

**Kết quả cần thấy:** video đầu ra, thường mang đuôi `.dub.mp4`, tại đường dẫn được ghi trong nhật ký. Đầu ra có thể nằm trong thư mục con theo video/lượt chạy; dùng đường dẫn nhật ký để tìm đúng bản vừa xuất.

## 4. Các cách sử dụng khác

### Đã có phụ đề SRT/ASS

1. Nếu cần xuất video, thêm và chọn video nguồn trước.
2. Trong **Dịch thuật**, chọn tệp và bấm **Tải phụ đề**. Tệp cần mã hóa UTF-8 (có thể có BOM).
3. Kiểm tra nội dung sau khi nạp: ứng dụng có lọc dòng nhiễu ngắn và điều chỉnh một số thời lượng; dùng hoàn tác nếu cần xem lại thay đổi.
4. Dịch nếu nội dung chưa ở ngôn ngữ mong muốn; nếu đã đúng, chuyển sang lồng tiếng hoặc xuất video.

SRT không giữ style/vị trí ASS. Dùng **Xuất SRT mới** khi muốn giữ bản nguồn riêng; lưu dự án để giữ thêm dữ liệu editor. Không cần đánh dấu bản dịch hoàn tất để lồng tiếng một tệp phụ đề hợp lệ.

### Chỉ tạo audio từ phụ đề hoặc văn bản

Trong **Lồng tiếng**, chọn SRT/ASS/TXT tại tệp đầu vào, chọn nơi xuất và giọng rồi bấm **Lồng tiếng** trong tab. SRT/ASS dùng mốc thời gian của phụ đề; TXT tạo giọng đọc từ nội dung văn bản. Không cần thêm video nếu chỉ xuất audio.

### Xử lý nhiều video hoặc nhiều tệp lồng tiếng

- Với video, chọn **Cả thư mục** hoặc **Chọn nhiều video**. Chọn thư mục chỉ lấy các video trực tiếp trong thư mục đó. Tick các thẻ cần xử lý; dùng **Chọn tất cả** nếu muốn xử lý cả danh sách.
- Kiểm tra vùng OCR và cấu hình trước khi chạy chung, đặc biệt khi các video có bố cục phụ đề khác nhau.
- Có thể chạy từng bước trên tập đã chọn hoặc dùng **Chạy pipeline** với **Dừng sau** ở bước mong muốn. Để pipeline chạy tới xuất video, cần chuẩn bị cấu hình nhận diện, dịch và giọng đọc trước.
- Với audio độc lập, bật **Lồng tiếng hàng loạt**, thêm các tệp SRT/ASS/TXT (chọn thư mục sẽ lấy SRT/ASS); kiểm tra danh sách và cấu hình ghi đè riêng từng tệp trước khi chạy. Danh sách này riêng với danh sách video trên preview.
- Dùng thư mục đầu ra riêng cho các nhóm tệp lồng tiếng có cùng tên để tránh nhầm kết quả.

### Gộp video

Ở chế độ nhiều video, thêm ít nhất hai video và bấm **Gộp videos**. Nếu tick ít nhất hai thẻ, ứng dụng gộp tập được tick; nếu không, gộp toàn bộ danh sách preview theo thứ tự hiện có. Kiểm tra thứ tự và nơi xuất trước khi chạy. Bật **Chuẩn hóa khi gộp** nếu muốn chuẩn hóa các đoạn; thiếu video nguồn sẽ được báo lỗi.

### Dịch theo ngữ cảnh

Trong **Cấu hình**, phần dịch theo ngữ cảnh hỗ trợ video đang chọn hoặc danh sách có phụ đề. Xem và sửa kết quả rồi bấm **Áp dụng vào phụ đề**. Kiểm tra tùy chọn lồng tiếng sau áp dụng nếu chỉ muốn sửa bản dịch. Khi bật sử dụng audio ngữ cảnh, âm thanh được gửi tới Gemini; cần mạng và tài khoản phù hợp.

## 5. Lưu dữ liệu và tiếp tục công việc

| Thao tác/dữ liệu | Ý nghĩa |
| --- | --- |
| **Lưu Dự Án** | Lưu JSON chứa video, vùng nhận diện, dữ liệu editor, bản nháp và liên kết công việc |
| **Mở Dự Án** | Khôi phục dữ liệu làm việc; không tự phát video hoặc chạy pipeline |
| **Lưu SRT / Xuất SRT mới** | Ghi tệp phụ đề có thể dùng bên ngoài ứng dụng |
| `settings.json` | Cấu hình dùng chung, API key, giọng mặc định, license và đường dẫn dự án gần nhất |
| Checkpoint/cache | Giữ tiến độ/kết quả hợp lệ để dùng lại khi chạy tiếp |

Sau khi đã mở/lưu dự án, ứng dụng tự lưu thay đổi của dự án. Nếu chưa chọn đường dẫn dự án, bản nháp chỉ ở bộ nhớ; hãy dùng **Lưu Dự Án** trước khi đóng. Khi đóng hoặc chuyển dự án chưa lưu, chọn Lưu/Bỏ thay đổi/Hủy theo nhu cầu.

Giữ **Lưu checkpoint OCR để tiếp tục sau lỗi / dừng** khi muốn phục hồi tiến độ. **Tạm dừng** yêu cầu tác vụ dừng ở điểm xử lý phù hợp; dùng **Tiếp tục** để chạy lại. **Dừng** hủy công việc hiện tại, hãy chờ tác vụ kết thúc trước khi chạy lượt khác.

Khi chạy lại với cùng đầu vào và cấu hình, ứng dụng dùng lại kết quả/checkpoint còn hợp lệ. Thay nội dung, model, vùng nhận diện hoặc cấu hình liên quan có thể làm kết quả cũ không còn phù hợp. Sau lỗi, kiểm tra dữ liệu đã lưu trước khi chọn trích xuất lại; thao tác trích xuất lại có thể thay kết quả đang dùng.

Không xóa thư mục đầu ra/cache nếu muốn tiếp tục công việc. Khi chuyển máy, mang theo dự án, video và các tệp kết quả liên quan; sửa lại đường dẫn bị mất và cấu hình API trên máy mới.

## 6. Lỗi thường gặp

| Hiện tượng | Cách kiểm tra và xử lý |
| --- | --- |
| Bị khóa xử lý hoặc quota hết | Kiểm tra **Cấu hình → License**, mạng, quota và hạn license; đọc trạng thái xác minh |
| Nút Dịch/Lồng tiếng không dùng được | Chọn đúng video, kiểm tra phụ đề đã nạp và chờ tác vụ hiện tại kết thúc; xem nhật ký nếu lưu dữ liệu thất bại |
| OCR thiếu hoặc sai chữ | Kiểm tra vùng quét qua nhiều khung hình, nguồn có đủ nét không; thử chế độ chính xác rồi kiểm tra/sửa phụ đề |
| ASR không có kết quả | Kiểm tra video có audio/lời thoại, ngôn ngữ, model và runtime; lần tải model đầu cần mạng |
| API trả 401/403 hoặc lỗi credentials | Kiểm tra key, tài khoản, quyền dùng model và URL API; lưu cấu hình rồi test lại |
| API trả 429 | Kiểm tra quota, giảm số tác vụ song song hoặc chờ cooldown; chỉ thêm profile/tài khoản có quota hợp lệ |
| GPU lỗi hoặc thiếu VRAM | Dùng **Kiểm tra**, thử CPU; với VieNeu GPU, kiểm tra bộ chạy/model và giảm batch nếu cần |
| Edge timeout/không có audio | Kiểm tra mạng và giọng, chờ rồi chạy lại; ứng dụng giữ cache hợp lệ và không tự đổi giọng/nhà cung cấp |
| Dòng lồng tiếng đỏ | Nghe đoạn đó; rút gọn lời, chỉnh thời gian hoặc Rate rồi chạy lại. Đây có thể là cảnh báo thời lượng dù audio vẫn tạo thành công |
| Xuất video lỗi | Kiểm tra FFmpeg, quyền ghi/dung lượng, video nguồn và ảnh logo; đọc lỗi cuối nhật ký |
| Không tìm thấy đầu ra | Xem đường dẫn hoàn tất trong nhật ký và các thư mục con theo video/lượt chạy |

Nhật ký trong giao diện cho biết bước đang chạy và lỗi cụ thể; các log chẩn đoán nằm trong `logs` cạnh ứng dụng. Khi cần hỗ trợ, ghi lại phiên bản, bước gây lỗi và thông báo liên quan; không gửi API key hoặc `settings.json` còn chứa khóa.

## 7. Chạy từ mã nguồn trên Windows

Nếu dùng bản EXE, có thể bỏ qua phần này. Trong PowerShell tại thư mục dự án:

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements.txt
.\.venv\Scripts\python scripts/setup_ffmpeg.py
.\.venv\Scripts\python main.py
```

Dự án đã được kiểm tra với Python 3.14. Script FFmpeg chuẩn bị công cụ riêng tại `.tools/ffmpeg`. Với ASR, chuẩn bị runtime trước khi dùng:

```powershell
.\.venv\Scripts\python scripts/prepare_asr_runtime.py
```

Xuất frame trên NVIDIA GPU từ source cần Visual Studio 2022 Build Tools với **Desktop development with C++ + Windows SDK**, sau đó chạy:

```powershell
.\.venv\Scripts\python scripts/setup_gpu_export.py
```

CPU không cần backend xuất GPU này. Xem [README](../README.md) cho cài đặt/build nâng cao và [tài liệu ASR](speech-recognition.md) cho runtime/alignment.

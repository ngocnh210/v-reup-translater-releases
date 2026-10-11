# Nhận diện giọng nói và đối chiếu phụ đề

Trong tab **Trích xuất**, chọn **OCR Local**, **Nhận diện giọng nói (ASR)** hoặc **ASR + OCR (đối chiếu)**. ASR dùng Faster-Whisper; FFmpeg chuyển âm thanh của đoạn đã chọn thành mono 16 kHz. CPU/GPU lấy từ khối **Xuất phụ đề**, ngôn ngữ lấy từ **Ngôn ngữ nguồn**; chọn tự động để Whisper phát hiện ngôn ngữ.

## Model, VAD và timestamp

Model mặc định `small`; có thể chọn `tiny`, `base`, `medium`, `large-v2`, `large-v3`, `large-v3-turbo`. Model tải ở lần sử dụng đầu vào `models/speech`; lần sau dùng cache. Khi tải thất bại, ứng dụng báo lỗi trong log. Các model chưa tải cần kết nối mạng.

**VAD** mặc định bật. Các khoảng không có tiếng nói được bỏ qua khi suy luận, không cắt video hoặc thay đổi timeline phụ đề. Timestamp luôn cộng lại mốc bắt đầu đoạn trích.

**Căn chỉnh timestamp (WhisperX)** mặc định tắt. Khi bật, ứng dụng dùng model alignment theo ngôn ngữ đã chọn/phát hiện. Bản WhisperX được khóa có model mặc định cho `zh`, `vi`, `en`, `ja`, `ko`; `th` chưa có model mặc định. Nếu ngôn ngữ không hỗ trợ, model không tải được hoặc alignment không thành công, ứng dụng giữ thời gian ASR và ghi cảnh báo. Alignment cũng cần tải model lần đầu; không đảm bảo khớp khẩu hình tuyệt đối.

## CPU/GPU

CPU dùng `int8`. ASR GPU dùng NVIDIA CUDA, kiểm tra kiểu tính toán mà thiết bị hỗ trợ. Khi lỗi CUDA hoặc thiếu VRAM, ứng dụng thử lại toàn bộ ASR bằng CPU một lần, không giữ những câu của lần GPU thất bại.

CTranslate2 4.6 cần các thư viện cuBLAS CUDA 12 và cuDNN 9. Helper tìm thư viện trong môi trường hệ thống và runtime VieNeu GPU đã cài tại `runtimes/vieneu-gpu`. GPU DirectML của OCR không thay thế CUDA cho ASR. Nút **Kiểm tra** hiển thị khả năng ASR và các ngôn ngữ có alignment.

Runtime mặc định chứa PyTorch CPU để giảm dung lượng. ASR vẫn dùng CUDA khi các thư viện NVIDIA khả dụng; alignment chạy CPU và thông báo nếu PyTorch CUDA không có. Muốn alignment trên GPU, chuẩn bị runtime bằng tùy chọn `--cuda` bên dưới. Mỗi công việc chạy một tác vụ ASR/alignment tại một thời điểm để giới hạn bộ nhớ model.

## ASR + OCR

OCR đọc phụ đề Trung trong vùng đã chọn; ASR nhận dạng lời thoại. Hai nguồn được ghép theo khoảng thời gian chồng nhau, hỗ trợ nhóm một–nhiều câu. So sánh bỏ qua khoảng trắng, dấu câu và khác biệt Unicode tương đương. Nội dung phải trùng sau chuẩn hóa mới được đánh dấu khớp; khác dù chỉ một chữ vẫn cần kiểm tra. Điểm từ 75% trở lên được ghi “Gần giống, cần kiểm tra”, tránh bỏ sót tên riêng hoặc chữ nhận diện sai.

- Câu khớp: giữ nội dung OCR, lấy khoảng thời gian lời thoại. Nếu nhóm gồm nhiều câu OCR, phân bố các mốc nội bộ theo tỷ lệ thời gian OCR.
- Câu khác nội dung: giữ OCR, đánh dấu cần kiểm tra.
- Chỉ có ASR: bổ sung câu, đánh dấu thiếu OCR.
- Chỉ có OCR: giữ câu, đánh dấu thiếu ASR.
- ASR khác tiếng Trung: giữ OCR, ghi khác ngôn ngữ; không kết luận nội dung sai hoặc tự thêm câu ngoại ngữ.

Nút **Đối chiếu ASR + OCR** mở bảng hai nguồn, điểm tương đồng và trạng thái. Có thể xem/nghe đoạn đã chọn, chọn OCR hoặc ASR để thay phụ đề trong nhóm câu. Thay đổi đi qua lịch sử của trình sửa phụ đề và có thể hoàn tác. Nếu nội dung đã sửa khác cả hai nguồn hoặc câu hiện tại vượt khoảng nhóm, ứng dụng từ chối ghi đè để bảo vệ chỉnh sửa.

Nếu một nguồn lỗi, ứng dụng dùng nguồn còn lại và cảnh báo đối chiếu chưa đầy đủ. Nếu cả hai nguồn lỗi, bước trích xuất thất bại. Video im lặng có thể tạo phụ đề rỗng hợp lệ; video không có audio là lỗi ASR, nhưng chế độ kết hợp vẫn có thể dùng OCR.

## Runtime và bản đóng gói

GUI tiếp tục dùng Python 3.14; inference chạy trong Python 3.12 riêng, không cài thư viện từ giao diện:

```powershell
.venv/Scripts/python.exe scripts/prepare_asr_runtime.py
# Tùy chọn: PyTorch CUDA 12.8 cho alignment GPU
.venv/Scripts/python.exe scripts/prepare_asr_runtime.py --cuda
# Khi build bản có PyTorch CUDA:
scripts/build_windows.ps1 -AsrCuda
```

Script kiểm tra SHA-256 của Python embedded, cài dependencies từ `requirements-asr.txt`, WhisperX 3.7.4 chỉ dùng alignment (không cài diarization), rồi kiểm tra import. Build Windows tự chuẩn bị runtime mặc định và đưa runtime vào cả portable và onefile. Các model Whisper/alignment không nằm trong bản phát hành; release check kiểm tra helper và registry alignment mà không tải model.

Kết quả giữ schema JSON phụ đề hiện có, xuất SRT/ASS/VTT và tiếp tục dùng trong dịch, lồng tiếng, xuất video. File phụ gồm `.asr.raw.json`, `.ocr.raw.json`, `.speech.compare.json` và `.speech.alignment.json`; metadata không bị trộn vào các dòng phụ đề. Khi bật checkpoint, bước ASR/OCR đã hoàn thành được tái sử dụng sau khi hủy; bước suy luận đang dở chạy lại. Thay model, ngôn ngữ, VAD, alignment hoặc chế độ sẽ tách cache.

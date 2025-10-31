# BÀI TẬP VỀ NHÀ – MÔN: AN TOÀN VÀ BẢO MẬT THÔNG TIN
Chủ đề: Chữ ký số trong file PDF
---
I. MÔ TẢ CHUNG
- Sinh viên thực hiện báo cáo và thực hành: phân tích và hiện thực việc nhúng, xác
thực chữ ký số trong file PDF.
- Phải nêu rõ chuẩn tham chiếu (PDF 1.7 / PDF 2.0, PAdES/ETSI) và sử dụng công cụ
thực thi (ví dụ iText7, OpenSSL, PyPDF, pdf-lib).
---
II. CÁC YÊU CẦU CỤ THỂ
1) Cấu trúc PDF liên quan chữ ký (Nghiên cứu)
- Mô tả ngắn gọn: Catalog, Pages tree, Page object, Resources, Content streams,
XObject, AcroForm, Signature field (widget), Signature dictionary (/Sig),
/ByteRange, /Contents, incremental updates, và DSS (theo PAdES).
- Liệt kê object refs quan trọng và giải thích vai trò của từng object trong
lưu/truy xuất chữ ký.
- Đầu ra: 1 trang tóm tắt + sơ đồ object (ví dụ: Catalog → Pages → Page → /Contents
; Catalog → /AcroForm → SigField → SigDict).
2) Thời gian ký được lưu ở đâu?
- Nêu tất cả vị trí có thể lưu thông tin thời gian:
+ /M trong Signature dictionary (dạng text, không có giá trị pháp lý).
+ Timestamp token (RFC 3161) trong PKCS#7 (attribute timeStampToken).
+ Document timestamp object (PAdES).
+ DSS (Document Security Store) nếu có lưu timestamp và dữ liệu xác minh.
- Giải thích khác biệt giữa thông tin thời gian /M và timestamp RFC3161.
3) Các bước tạo và lưu chữ ký trong PDF (đã có private RSA)
- Viết script/code thực hiện tuần tự:
1. Chuẩn bị file PDF gốc.
2. Tạo Signature field (AcroForm), reserve vùng /Contents (8192 bytes).
3. Xác định /ByteRange (loại trừ vùng /Contents khỏi hash).
4. Tính hash (SHA-256/512) trên vùng ByteRange.
5. Tạo PKCS#7/CMS detached hoặc CAdES:
- Include messageDigest, signingTime, contentType.
- Include certificate chain.
- (Tùy chọn) thêm RFC3161 timestamp token.
6. Chèn blob DER PKCS#7 vào /Contents (hex/binary) đúng offset.
7. Ghi incremental update.
8. (LTV) Cập nhật DSS với Certs, OCSPs, CRLs, VRI.
- Phải nêu rõ: hash alg, RSA padding, key size, vị trí lưu trong PKCS#7.
- Đầu ra: mã nguồn, file PDF gốc, file PDF đã ký.4) Các bước xác thực chữ ký trên PDF đã ký
- Các bước kiểm tra:
1. Đọc Signature dictionary: /Contents, /ByteRange.
2. Tách PKCS#7, kiểm tra định dạng.
3. Tính hash và so sánh messageDigest.
4. Verify signature bằng public key trong cert.
5. Kiểm tra chain → root trusted CA.
6. Kiểm tra OCSP/CRL.
7. Kiểm tra timestamp token.
8. Kiểm tra incremental update (phát hiện sửa đổi).
- Nộp kèm script verify + log kiểm thử.
---
III. YÊU CẦU NỘP BÀI
1. Báo cáo PDF ≤ 6 trang: mô tả cấu trúc, thời gian ký, rủi ro bảo mật.
2. Code + README (Git repo hoặc zip).
3. Demo files: original.pdf, signed.pdf, tampered.pdf.
4. (Tuỳ chọn) Video 3–5 phút demo kết quả.
---
# Bài Làm
### 1. Sinh khóa RSA và chứng thư số
File: scripts/gen_keys.py
Thực hiện:

`cd scripts
python gen_keys.py`

Kết quả:
- keys/signer_key.pem: Khóa riêng (RSA 2048-bit)
- keys/signer_cert.pem: Chứng thư số tự ký (self-signed)
- Mục đích: Dùng để ký và xác thực chữ ký.
- Thuật toán: RSA 2048-bit, SHA-256, padding PKCS#1 v1.5

### 2. Tạo và ký file PDF
- File gốc chưa ký
<img width="1920" height="1080" alt="Screenshot (37)" src="https://github.com/user-attachments/assets/168ed678-4168-4ef0-8103-6c89838aedc5" />

- File đã ký
<img width="1920" height="1080" alt="Screenshot (38)" src="https://github.com/user-attachments/assets/bad61517-bb30-4556-bd2f-ee0878db3b84" />

- Phần báo giả mạo đang bị lỗi, không tiện show lên.
<img width="1920" height="1080" alt="Screenshot (39)" src="https://github.com/user-attachments/assets/6407890f-d53c-4cda-9984-b78a6b7a6a6b" />

# Sẽ sửa lỗi vào cài đặt lại sớm nhất có thể

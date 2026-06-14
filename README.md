# Tra cứu điểm thi

Trang web tĩnh tra cứu điểm thi theo **Mã định danh**, dữ liệu lấy trực tiếp từ Google Sheet.

## 1. Chuẩn bị Google Sheet

1. Mở Google Sheet → **Chia sẻ (Share)** → đổi quyền thành **"Anyone with the link – Viewer"** (Bất kỳ ai có liên kết – Người xem). Bắt buộc, nếu không server sẽ không đọc được dữ liệu.
2. Ghi nhớ:
   - **Sheet ID**: phần chuỗi dài trong URL, ví dụ `1sdUo3LSKAk82Sqo8jZsmUeqZ2SDWyOofbRkJRiM20hE`.
   - **Tên tab** chứa dữ liệu (ví dụ `Trang tính1`).
3. File `api/scores.js` đã được cấu hình sẵn:
   ```js
   const SHEET_ID = "1sdUo3LSKAk82Sqo8jZsmUeqZ2SDWyOofbRkJRiM20hE";
   const SHEET_NAME = "Sheet1";
   ```
   Nếu tên tab trong Sheet của bạn khác `Sheet1`, sửa lại giá trị này.

Cấu trúc cột cần có (đúng thứ tự hoặc tên gần giống):
`STT | Ma dinh danh | Ho va ten | Diem Toan | Diem TV | Diem tieng Anh | Tong diem`

Code tự nhận diện cột theo tên, nên có thể đổi vị trí cột miễn tên tương tự.

## Cấu trúc project

```
diem-thi/
├── index.html       # trang tra cứu (gọi /api/scores)
├── api/
│   └── scores.js    # Vercel Serverless Function, đọc Google Sheet + cache
└── README.md
```

## 2. Đưa lên GitHub

```bash
git init
git add .
git commit -m "Trang tra cứu điểm thi"
git branch -M main
git remote add origin <link-repo-github-cua-ban>
git push -u origin main
```

## 3. Deploy lên Vercel

1. Vào [vercel.com](https://vercel.com) → **New Project** → chọn repo vừa push.
2. Không cần cấu hình build (đây là static site, Framework = "Other").
3. Deploy.

## 4. Về khả năng chịu tải ~2.000 người truy cập đồng thời

Project này đã được thiết kế sẵn theo hướng an toàn cho lượng truy cập lớn:

- **File tĩnh (`index.html`)**: Vercel CDN phục vụ rất tốt, 2.000 người dùng đồng thời không phải vấn đề.
- **Dữ liệu điểm**: client KHÔNG gọi trực tiếp Google Sheets, mà gọi `/api/scores` (Vercel Serverless Function).
  - Function này có header `Cache-Control: s-maxage=600, stale-while-revalidate=3600` → Vercel CDN cache kết quả trong 10 phút.
  - Trong 10 phút đó, **dù có hàng chục nghìn người truy cập, Google Sheet chỉ bị gọi khoảng 1 lần** (mỗi region edge của Vercel gọi lại khi cache hết hạn).
  - Nếu Google Sheet phản hồi lỗi/chậm, CDN vẫn phục vụ bản cache cũ tối đa 1 giờ (`stale-while-revalidate`).

→ Với cấu trúc này, 2.000 người truy cập cùng lúc hoàn toàn ổn, không lo bị Google rate-limit.

**Lưu ý khi điểm có thay đổi**: do cache 10 phút, nếu cập nhật điểm trong Sheet thì người dùng có thể thấy dữ liệu cũ trong tối đa 10 phút. Có thể giảm `s-maxage` xuống (ví dụ 60–120 giây) nếu cần cập nhật nhanh hơn, miễn vẫn còn đệm để tránh gọi Google quá thường xuyên.

**Phương án thay thế** (nếu muốn đơn giản hơn, không cần function): export dữ liệu Sheet ra file `data.json` tĩnh, đặt cùng repo, mỗi lần có điểm mới thì export lại và deploy. Phù hợp khi điểm đã chốt và không thay đổi thường xuyên.

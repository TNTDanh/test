# Hangry

Hangry là nền tảng đặt đồ ăn đa kênh gồm web cho khách hàng, portal quản trị, console quản lý user, ứng dụng di động (Expo) và backend Express/MongoDB. Hệ thống hỗ trợ quản lý thực đơn, giỏ hàng, đặt COD/Stripe và theo dõi đơn hàng gần thời gian thực.

## Cấu trúc thư mục

| Đường dẫn        | Mô tả                                                                 | Công nghệ chính                    |
| ---------------- | --------------------------------------------------------------------- | ---------------------------------- |
| `backend/`       | REST API, xác thực, CRUD món, giỏ hàng, đơn, thanh toán, upload ảnh  | Node.js, Express, MongoDB, Stripe  |
| `frontend/`      | Website khách đặt món (Vite + React)                                 | React 18, Vite, Axios              |
| `admin/`         | Portal vận hành: thêm món, xem menu, cập nhật trạng thái đơn         | React 18, React Router, Toastify   |
| `users-admin/`   | Console nhẹ quản lý người dùng (tìm kiếm, đổi role, khóa)           | React 18, React Router             |
| `mobile/`        | Ứng dụng Expo: tab Home, Cart, Orders, Profile                       | Expo Router, React Native, Zustand |
| `uploads/`       | Thư mục chứa ảnh khi không dùng Cloudinary (serve qua `/images/*`)   | Static assets của Express          |

## Tính năng chính

- **Backend thống nhất**: JWT auth, lưu giỏ, CRUD menu, Stripe checkout (`/api/order/place`) & COD (`/api/order/place-cod`), upload Multer + Cloudinary.
- **Web khách hàng**: duyệt danh mục, lọc, thêm/giảm giỏ, đăng nhập/đăng ký (`LoginPopup`), đặt đơn, xác minh thanh toán `/verify`.
- **Admin portal**: thêm món có kiểm tra dữ liệu, danh sách món, cập nhật trạng thái đơn (Food Processing → Delivered).
- **User console**: tìm kiếm, lọc role/trạng thái, chuyển user ↔ admin, khóa/mở và xóa người dùng.
- **App di động**: tab-based, quản lý giỏ, đặt hàng, đăng nhập, xem lịch sử; tái sử dụng cùng API.
- **Quản lý media**: ảnh địa phương qua `/images/<file>` hoặc URL Cloudinary.

## Công nghệ

- **Backend:** Node 20+, Express 4, Mongoose 8, JWT, bcrypt, Multer, Stripe, Cloudinary (tùy chọn).
- **Web:** React 18, Vite 5/7, React Router 6, Axios, React Icons/Toastify.
- **Mobile:** Expo SDK 54, React Native 0.81, Expo Router, React Query, Zustand, AsyncStorage.
- **Tooling:** Nodemon, ESLint, Vite, Expo CLI.

## Chuẩn bị

- Node.js ≥ 20 và npm ≥ 10.
- MongoDB Atlas hoặc local.
- Khóa Stripe test.
- (Tuỳ chọn) Cloudinary.
- Expo CLI + thiết bị/simulator.

## Cài đặt

```bash
git clone https://github.com/TNTDanh/Software_Hangry.git
cd Software_Hangry

cd backend && npm install
cd ../frontend && npm install
cd ../admin && npm install
cd ../users-admin && npm install
cd ../mobile && npm install
Biến môi trường
backend/.env

PORT=4000
MONGO_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/hangry
JWT_SECRET=<chuỗi-bất-kỳ>
CLIENT_URL=http://localhost:5173,http://localhost:4173,http://localhost:5174
FRONTEND_URL=http://localhost:5173
STRIPE_SECRET_KEY=sk_test_xxx
CLOUDINARY_CLOUD_NAME=<nếu dùng>
CLOUDINARY_API_KEY=<nếu dùng>
CLOUDINARY_API_SECRET=<nếu dùng>
CLIENT_URL: danh sách origin (cách nhau dấu phẩy) được CORS cho phép.
Bỏ trống Cloudinary nếu muốn lưu file trong uploads/.
frontend/.env

VITE_API_URL=http://localhost:4000
admin/.env (hoặc .env.local)

VITE_API_URL=http://localhost:4000
users-admin/.env

VITE_API_URL=http://localhost:4000
mobile
Trong mobile/src/api/client.ts và mobile/src/until/image.ts sửa API_URL trỏ về backend (VD: http://10.0.2.2:4000 cho Android emulator). Có thể refactor đọc process.env.EXPO_PUBLIC_API_URL nếu muốn cấu hình động.

Chạy các dịch vụ
Dịch vụ	Lệnh	Ghi chú
Backend API	cd backend && npm run server	Nodemon, port 4000 (hoặc PORT).
Web khách hàng	cd frontend && npm run dev	Mặc định 5173.
Admin portal	cd admin && npm run dev	Cần VITE_API_URL.
User console	cd users-admin && npm run dev	Redirect tới /users.
Mobile (Expo)	cd mobile && npx expo start	Dùng npm run android/ios/web tùy môi trường.
Luôn khởi động backend trước để các client gọi API thành công.

API chính
Method	Endpoint	Mô tả	Header token
POST	/api/user/register	Đăng ký (name, email, password).	Không
POST	/api/user/login	Đăng nhập, trả JWT.	Không
GET	/api/user/list	Danh sách/tìm kiếm user (console).	Nên bảo vệ
POST	/api/user/update	Đổi role hoặc trạng thái hoạt động.	Nên bảo vệ
POST	/api/user/remove	Xóa user.	Nên bảo vệ
POST	/api/food/add	Thêm món + upload ảnh.	Admin
GET	/api/food/list	Lấy toàn bộ menu.	Không
POST	/api/food/remove	Xóa món (dọn ảnh local/Cloudinary).	Admin
POST	/api/cart/add	Tăng số lượng 1 món trong giỏ.	Có
POST	/api/cart/remove	Giảm số lượng.	Có
POST	/api/cart/get	Lấy giỏ lưu trên server.	Có
POST	/api/order/place	Stripe Checkout, trả session_url.	Có
POST	/api/order/place-cod	Đặt COD.	Có
POST	/api/order/verify	FE gọi sau khi Stripe redirect về /verify.	Không
POST	/api/order/userorders	Lịch sử đơn của user hiện tại.	Có
GET	/api/order/list	Admin xem tất cả đơn.	Admin
POST	/api/order/status	Cập nhật trạng thái đơn.	Admin
Ảnh local nằm trong uploads/ và được serve qua /images/<tên_file>. Khi dùng Cloudinary, DB lưu luôn URL CDN.

Quy trình Stripe/COD
Frontend/mobile gửi items, amount, address tới /api/order/place.
Backend tạo bản ghi order (payment=false) và session Stripe.
Người dùng thanh toán trên Stripe, sau đó được redirect về ${FRONTEND_URL}/verify?success=....
FE gọi /api/order/verify với orderId, success.
COD dùng /api/order/place-cod, không cần Stripe.
Đảm bảo FRONTEND_URL trùng port đang chạy trang /verify.

Ứng dụng di động
Tabs nằm trong mobile/app/(tabs) (index, cart, orders, myorders, profile).
State dùng Zustand (mobile/lib/store/cart.ts, mobile/lib/store/auth.ts).
Axios client tự gắn header token từ store.
Nhớ chỉnh API_URL phù hợp thiết bị (Android emulator dùng 10.0.2.2, iOS simulator có thể dùng localhost).
Lưu ý phát triển
Nên seed sẵn dữ liệu food trong Mongo để UI có món hiển thị.
Thư mục uploads/ phải được ghi bởi process Node.
Khi thêm origin mới (VD: admin chạy port khác) phải cập nhật CLIENT_URL.
Các route admin (user, menu, order) hiện chưa có middleware; cân nhắc bổ sung nếu deploy production.
Đóng góp
Fork repo.
Tạo nhánh: git checkout -b feat/<ten-tinh-nang>.
Commit & test.
Mở Pull Request về nhánh dev.
License
MIT



Gợi ý tiếp theo:  
1. Đặt file `.env` và cập nhật `API_URL` trong app mobile để chạy local.  
2. Kiểm tra lại các route admin cần middleware bảo vệ trước khi đưa vào môi trường thực tế.

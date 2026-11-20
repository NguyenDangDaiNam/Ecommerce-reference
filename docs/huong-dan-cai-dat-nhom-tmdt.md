# Chạy Ecommerce-reference cùng nhóm TMĐT

## Lấy đúng bản sao

Trong GitHub Desktop, đăng nhập bằng tài khoản GitHub của bạn. Nếu repo còn private, chấp nhận lời mời của chủ repo trước khi clone.

- **Lần đầu** chọn **File → Clone Repository → URL**, nhập `https://github.com/NguyenDangDaiNam/Ecommerce-reference.git` và chọn thư mục local. [Hướng dẫn GitHub Desktop](https://docs.github.com/en/desktop/adding-and-cloning-repositories/cloning-and-forking-repositories-from-github-desktop)
- **Đã clone** chọn đúng repo trong GitHub Desktop rồi dùng **Fetch origin** và **Pull origin** khi có cập nhật. Chọn **Repository → Open in Visual Studio Code** để mở folder.

## Chạy và dừng

Windows và macOS dùng chung quy trình ở [README](../README.md#chạy-trên-windows-hoặc-macos). Cài Docker Desktop một lần. Sau khi mở Docker Desktop, chạy hai lệnh này từ thư mục gốc repo:

| Việc cần làm | Lệnh |
|---|---|
| Khởi động hoặc cập nhật sau khi pull | `docker compose up --build --wait` |
| Dừng, giữ dữ liệu local | `docker compose stop` |

Mở [OldShop](http://127.0.0.1:8080) sau khi lệnh khởi động hoàn tất. Mỗi máy có database mẫu riêng. Dùng tài khoản thử trong README để kiểm tra đăng nhập, giỏ hàng và checkout COD.

Khi sửa mã, tạo branch làm việc trong GitHub Desktop. Trước khi push hoặc mở pull request, kiểm tra đích là `NguyenDangDaiNam/Ecommerce-reference`. Giữ file `.env` và dữ liệu thử trên máy.

# BÁO CÁO BÀI TẬP

## 1. Cấu trúc Kịch bản CI/CD (`.github/workflows/ci.yml`)

Workflow gồm 2 giai đoạn:
1. **Job `test`:**
   - Khởi tạo JDK 17 (Temurin).
   - Chạy lệnh `./gradlew test bootJar`.
2. **Job `docker-build`:**
   - Khai báo ràng buộc `needs: test`.
   - Cấp quyền `packages: write` cho `GITHUB_TOKEN`.
   - Sử dụng `docker/login-action@v3` để đăng nhập vào `ghcr.io`.
   - Dùng `docker/build-push-action@v5` đóng gói và đẩy container image.

---

## 2. Kết quả Nghiệm thu (Verification)

### a. Sơ đồ liên kết 2 Job trên giao diện GitHub Actions (Kịch bản thành công)
- Khi mã nguồn vượt qua toàn bộ bài kiểm thử, mũi tên điều phối kích hoạt job `docker-build`:
  
```text
[ Run Gradle Test & Build JAR ]  (Success - Màu xanh)
               │
               ▼
[ Build and Push Docker Image ]   (Success - Màu xanh)
```



### b. Kiểm chứng kịch bản ngắt mạch khi Test thất bại (Skipped Condition)
- Khi có bài test JUnit bị đứt (`AssertionError`):
  - Job `test` dừng lại ở trạng thái **Failed** (Màu đỏ).
  - Do có chỉ thị `needs: test`, Job `docker-build` tự động chuyển sang trạng thái **Skipped** (Màu xám), ngăn chặn việc xuất xưởng Docker image bị lỗi ra Registry.

---

## 3. Kết luận
- Pipeline đảm bảo nguyên tắc CI/CD cốt lõi: **Không xuất bản artifact khi kiểm thử chất lượng chưa đạt yêu cầu**.
- Quá trình đăng nhập và push image lên `ghcr.io` sử dụng trực tiếp secret an toàn tích hợp sẵn `secrets.GITHUB_TOKEN`.
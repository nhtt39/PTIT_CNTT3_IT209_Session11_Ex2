# Session 11 - Exercise 2: Biên dịch đóng gói tệp ứng dụng .jar với Gradle và lưu trữ Artifacts

## 1. Mục tiêu bài thực hành

- Thực thi câu lệnh biên dịch và đóng gói ứng dụng Spring Boot thành tệp thực thi `.jar` bằng Gradle Wrapper: `./gradlew bootJar`.
- Lưu trữ thành phẩm biên dịch (`.jar`) sinh ra từ thư mục `build/libs/` lên GitHub Actions bằng action chính thức `actions/upload-artifact@v4`.
- Đặt tên cho artifact lưu trữ là `app-jar`.
- Kiểm tra kết quả trong phần **Summary** của workflow run trên tab **Actions**, xác nhận artifact xuất hiện và có thể tải về máy tính.

---

## 2. Cấu trúc thư mục dự án (`ex2_ss11`)

```text
ex2_ss11/
├── .github/
│   └── workflows/
│       └── ci.yml               # Kịch bản GitHub Actions CI (Test, bootJar & upload-artifact)
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar   # Gradle Wrapper JAR
│       └── gradle-wrapper.properties
├── src/
│   ├── main/
│   │   ├── java/com/example/demo/
│   │   │   └── DemoApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/example/demo/
│           └── DemoApplicationTests.java # JUnit test
├── .gitattributes               # Đảm bảo gradlew sử dụng LF line ending
├── .gitignore                   # Loại trừ các file build, cache
├── build.gradle                 # Cấu hình Spring Boot và Gradle build
├── gradlew                      # Script chạy Gradle Wrapper trên Linux/macOS
├── gradlew.bat                  # Script chạy Gradle Wrapper trên Windows
├── settings.gradle              # Cấu hình tên project
├── README.md                    # Tài liệu hướng dẫn chi tiết
└── images/                      # Thư mục chứa ảnh chụp minh chứng
```

---

## 3. Chi tiết tệp kịch bản GitHub Actions (`.github/workflows/ci.yml`)

Kịch bản được cấu hình để tự động kích hoạt khi có sự kiện `push` lên nhánh `main`:

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests & Package JAR
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar
```

### Phân tích chi tiết các bước trong Job:

1. **Checkout mã nguồn (`actions/checkout@v5`)**:
   - Tải toàn bộ mã nguồn của dự án về máy ảo runner `ubuntu-latest`.

2. **Thiết lập môi trường Java JDK 17 (`actions/setup-java@v5`)**:
   - Cài đặt Java JDK 17 phân phối bởi Eclipse Temurin (Adoptium).
   - Tự động cấu hình biến môi trường `JAVA_HOME` và đường dẫn `PATH`.

3. **Cấp quyền cho Gradle Wrapper (`chmod +x gradlew`)**:
   - Đảm bảo tệp thực thi wrapper có quyền chạy (`execute`) trên hệ điều hành Linux runner.

4. **Thực thi JUnit Test (`./gradlew test`)**:
   - Biên dịch và chạy toàn bộ các bài kiểm thử JUnit 5 có trong dự án.

5. **Đóng gói tệp JAR với Gradle (`./gradlew bootJar`)**:
   - Thực thi tác vụ `bootJar` của plugin Spring Boot Gradle.
   - Biên dịch toàn bộ mã nguồn ứng dụng và đóng gói thành tệp JAR độc lập (executable fat JAR) tại thư mục `build/libs/demo-0.0.1-SNAPSHOT.jar`.

6. **Lưu trữ thành phẩm JAR Artifact (`actions/upload-artifact@v4`)**:
   - Tải tệp JAR vừa được tạo ra từ đường dẫn `build/libs/*.jar` lên kho lưu trữ Artifact của GitHub Actions.
   - Đặt tên hiển thị cho artifact là `app-jar`.

---

## 4. Các bước khởi tạo Git và Đẩy mã nguồn lên GitHub

Mở PowerShell tại thư mục `ex2_ss11` và thực hiện tuần tự các lệnh sau:

```bash
# 1. Khởi tạo repository Git tại thư mục ex2_ss11
git init -b main

# 2. Cấp cờ quyền thực thi (executable 100755) cho gradlew trên môi trường Linux
git update-index --chmod=+x gradlew

# 3. Thêm toàn bộ các tệp vào staging
git add .

# 4. Thực hiện commit đầu tiên
git commit -m "feat: complete exercise 2 - build bootJar and upload artifact app-jar"

# 5. Liên kết với remote repository trên GitHub
# (Thay URL bên dưới bằng URL repository của bạn, ví dụ PTIT_CNTT3_IT209_Session11_Ex2)
git remote add origin https://github.com/nhtt39/PTIT_CNTT3_IT209_Session11_Ex2.git

# 6. Đẩy mã nguồn lên nhánh main
git push -u origin main
```

---

## 5. Kiểm tra kết quả trên GitHub Actions

1. Mở trình duyệt và truy cập vào repository trên GitHub của bạn.
2. Điều hướng đến tab **Actions**.
3. Bạn sẽ thấy quy trình **Java Spring Boot CI (Gradle)** được kích hoạt tự động từ commit vừa đẩy lên.
4. Nhấp vào lần chạy gần nhất để kiểm tra:
   - Các bước **Thực thi JUnit Test với Gradle**, **Đóng gói tệp JAR với Gradle**, **Lưu trữ thành phẩm JAR Artifact** đều hoàn thành với biểu tượng tích xanh (**Success** / `✓`).
5. Quay lại trang **Summary** của lần chạy (ở menu bên trái chọn **Summary**):
   - Cuộn xuống phần **Artifacts**.
   - Kiểm tra thấy mục **`app-jar`** xuất hiện kèm kích thước tệp và số ngày lưu trữ còn lại.
   - Bạn có thể nhấp chuột vào `app-jar` để tải tệp `.zip` về máy, giải nén và kiểm tra tệp `.jar` được đóng gói bên trong.

---

## 6. Hướng dẫn nộp bài

- **Sản phẩm nộp bài:**
  1. Tệp kịch bản [`.github/workflows/ci.yml`](.github/workflows/ci.yml) đã hoàn thiện.
  2. Ảnh chụp màn hình trang **Summary** trên tab GitHub Actions hiển thị luồng chạy hoàn thành thành công và hiển thị rõ mục **Artifacts** có chứa **`app-jar`**.
  3. Đặt ảnh chụp màn hình vào thư mục [`images/`](images/) (ví dụ: `images/summary_artifact.png`).

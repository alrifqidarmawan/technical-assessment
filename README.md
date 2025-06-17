# Technical Assessment: End-to-End CI/CD for Spring Boot Application

Proyek ini merupakan implementasi dari sebuah *technical assessment* yang bertujuan untuk mendemonstrasikan pemahaman tentang proses CI/CD (Continuous Integration/Continuous Deployment) dari awal hingga akhir. Aplikasi yang digunakan adalah sebuah service backend sederhana yang dibangun menggunakan **Java Spring Boot**.

Tujuan utama dari proyek ini adalah:
1.  Membuat aplikasi yang siap di-container-isasi dengan **Docker**.
2.  Mendesain alur CI/CD yang ideal dan otomatis.
3.  Mengimplementasikan alur tersebut menggunakan **Jenkinsfile**.
4.  Men-deploy aplikasi ke platform orkestrasi **Kubernetes**.

---

## 1. Spesifikasi Proyek

Aplikasi ini adalah sebuah REST API sederhana yang memiliki satu endpoint:
*   `GET /`: Mengembalikan pesan sambutan, misalnya `Welcome to Technical Assessment!`.

### Teknologi yang Digunakan
*   **Bahasa Pemrograman**: Java 21
*   **Framework**: Spring Boot 3.x
*   **Build Tool**: Maven
*   **Containerization**: Docker
*   **CI/CD Automation**: Jenkins
*   **Deployment Platform**: Kubernetes

---

## 2. Desain Alur CI/CD

Alur CI/CD ini dirancang untuk mencapai otomatisasi penuh dari saat kode dikirim oleh developer hingga aplikasi berjalan di lingkungan produksi, dengan tetap menjaga kualitas dan stabilitas.

### Diagram Alur (Konseptual)
`Developer` -> `Git Push (GitHub)` -> `Webhook` -> `Jenkins` -> `Build & Test` -> `Build Docker Image` -> `Push to Docker Hub` -> `Deploy to Kubernetes`

### Tahapan Alur Secara Detail

1.  **Code Commit & Push**
    *   Developer menulis kode pada *feature branch* dan melakukan `git push` ke repositori di GitHub.

2.  **Pull Request & CI Trigger**
    *   Developer membuat *Pull Request (PR)* ke *branch* `main`.
    *   Aksi ini secara otomatis memicu *webhook* yang menjalankan **Jenkins Pipeline**.

3.  **Continuous Integration (CI) Stage in Jenkins**
    *   **Checkout**: Jenkins mengambil kode sumber dari *branch* PR.
    *   **Unit & Integration Test**: Jenkins menjalankan perintah `mvn test` untuk memastikan semua tes otomatis berhasil. Jika ada tes yang gagal, *pipeline* akan berhenti dan memberikan notifikasi kegagalan.
    *   **Code Quality Analysis (Opsional)**: Idealnya, Jenkins akan mengintegrasikan **SonarQube** untuk memindai kode, mencari *bugs*, kerentanan, dan *code smells*. PR akan diblokir jika tidak memenuhi standar kualitas.
    *   **Build Application**: Jika semua tes dan analisis berhasil, Jenkins akan meng-compile kode dan mem-package aplikasi menjadi sebuah file `.jar` menggunakan `mvn clean package`.

4.  **Build & Push Docker Image**
    *   Jenkins menggunakan `Dockerfile` yang ada di dalam repositori untuk membangun sebuah Docker Image.
    *   Image tersebut akan diberi *tag* yang unik, misalnya menggunakan `BUILD_ID` dari Jenkins (contoh: `alrifqidarmawan/app-technical-assessment:v1`).
    *   Image yang sudah di-build kemudian di-*push* ke *image registry* seperti **Docker Hub**.

5.  **Continuous Deployment (CD) Stage**
    *   **Deploy to Staging**: Setelah PR di-*merge* ke `main`, Jenkins secara otomatis men-deploy image baru ke *environment* **Staging** di Kubernetes. Ini dilakukan dengan menerapkan file manifest (`deployment.yaml`, `service.yaml`).
    *   **Manual Approval Gate**: Untuk mencegah *deployment* yang tidak diinginkan ke produksi, *pipeline* akan berhenti dan menunggu persetujuan manual dari seorang QA Lead atau Project Manager.
    *   **Deploy to Production**: Setelah mendapatkan persetujuan, Jenkins akan men-deploy image yang sama yang telah teruji di Staging ke *environment* **Production** di Kubernetes. Proses ini menggunakan strategi *rolling update* untuk memastikan tidak ada *downtime*.

---

## 3. Cara Menjalankan Proyek Secara Lokal

### Prasyarat
*   Java JDK 21 atau lebih tinggi
*   Maven 3.x
*   Docker Desktop

### Menjalankan Aplikasi Langsung
```bash
# Clone repositori ini
git clone https://github.com/alrifqidarmawan/technical-assessment.git
cd technical-assessment

# Jalankan aplikasi menggunakan Maven
mvn spring-boot:run
```
Aplikasi akan berjalan di `http://localhost:8080`.

### Menjalankan Menggunakan Docker
1.  **Build Docker Image:**
    ```bash
    # Pastikan Anda berada di root direktori proyek
    docker build -t alrifqidarmawan/app-technical-assessment:latest .
    ```
2.  **Jalankan Container:**
    ```bash
    docker run -p 8080:8080 alrifqidarmawan/app-technical-assessment:latest
    ```
Akses aplikasi melalui `http://localhost:8080` di browser Anda.

---
*Catatan: File `Jenkinsfile` dan manifest Kubernetes (`k8s/`) ada di dalam repositori ini sebagai implementasi dari desain di atas.*
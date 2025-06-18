      
# Technical Assessment: End-to-End CI/CD for Spring Boot Application

![Java](https://img.shields.io/badge/Java-21-blue?style=for-the-badge&logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-green?style=for-the-badge&logo=spring)
![Docker](https://img.shields.io/badge/Docker-blue?style=for-the-badge&logo=docker)
![Kubernetes](https://img.shields.io/badge/Kubernetes-blue?style=for-the-badge&logo=kubernetes)
![Jenkins](https://img.shields.io/badge/Jenkins-gray?style=for-the-badge&logo=jenkins)

Proyek ini merupakan implementasi dari sebuah *technical assessment* yang mendemonstrasikan pemahaman tentang proses CI/CD (Continuous Integration/Continuous Deployment) dari awal hingga akhir. Aplikasi yang digunakan adalah sebuah service backend sederhana yang dibangun menggunakan **Java Spring Boot**.

Tujuan utamanya adalah membuat aplikasi yang siap di-container-isasi, mendesain alur CI/CD, dan men-deploy-nya ke platform orkestrasi **Kubernetes**.

---

## 📖 Daftar Isi

1.  [⚙️ Tumpukan Teknologi (Tech Stack)](#-tumpukan-teknologi-tech-stack)
2.  [📁 Struktur Proyek](#-struktur-proyek)
3.  [🌊 Desain Alur CI/CD](#-desain-alur-cicd)
4.  [🚀 Panduan Deployment Lokal](#-panduan-deployment-lokal)
5.  [🌐 Endpoint API](#-endpoint-api)
6.  [🧹 Membersihkan Environment](#-membersihkan-environment)

---

## ⚙️ Tumpukan Teknologi (Tech Stack)

*   **Bahasa Pemrograman**: Java 21
*   **Framework**: Spring Boot 3.x
*   **Build Tool**: Maven
*   **Containerization**: Docker
*   **CI/CD Automation**: Jenkins
*   **Deployment Platform**: Kubernetes (Minikube untuk lokal)

---

## 📁 Struktur Proyek

```
.
├── k8s/                    # Manifest Kubernetes (Deployment, Service, Ingress)
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── src/                    # Kode sumber aplikasi Spring Boot
├── .gitignore
├── Dockerfile              # Resep untuk membangun image Docker aplikasi
├── Jenkinsfile             # Definisi pipeline CI/CD untuk Jenkins
├── pom.xml                 # Konfigurasi proyek Maven
└── README.md               # Anda sedang membacanya
```

---

## 🌊 Desain Alur CI/CD

Alur CI/CD ini dirancang untuk mencapai otomatisasi penuh dari saat kode dikirim oleh developer hingga aplikasi berjalan di lingkungan produksi.

### Diagram Alur (Konseptual)
`Developer` → `Git Push (GitHub)` → `Webhook` → `Jenkins` → `Build & Test` → `Build & Push Docker Image` → `Deploy to Kubernetes`

### Tahapan Alur Secara Detail

1.  **Code Commit & Push**: Developer menulis kode pada *feature branch* dan melakukan `git push` ke repositori.
2.  **Pull Request & CI Trigger**: Developer membuat *Pull Request (PR)* ke *branch* `main`. Aksi ini secara otomatis memicu **Jenkins Pipeline**.
3.  **Continuous Integration (CI) Stage**:
    *   **Checkout**: Jenkins mengambil kode sumber.
    *   **Unit & Integration Test**: Menjalankan `mvn test`. Jika gagal, pipeline berhenti.
    *   **Code Quality Analysis (Opsional)**: Integrasi dengan SonarQube untuk memindai kode.
    *   **Build Application**: Jika semua tes berhasil, Jenkins mem-package aplikasi menjadi file `.jar` menggunakan `mvn clean package`.
4.  **Build & Push Docker Image**:
    *   Jenkins menggunakan `Dockerfile` untuk membangun image Docker.
    *   Image diberi *tag* unik (misal: dengan `BUILD_ID`).
    *   Image di-*push* ke *image registry* **Docker Hub**.
5.  **Continuous Deployment (CD) Stage**:
    *   **Deploy to Staging**: Setelah PR di-*merge* ke `main`, Jenkins secara otomatis men-deploy image baru ke environment Staging di Kubernetes.
    *   **Manual Approval Gate**: Pipeline berhenti dan menunggu persetujuan manual untuk rilis ke produksi.
    *   **Deploy to Production**: Setelah disetujui, Jenkins men-deploy image yang sama ke environment Produksi menggunakan strategi *rolling update* untuk memastikan *zero downtime*.

---

## 🚀 Panduan Deployment Lokal

Berikut adalah cara untuk men-deploy aplikasi ini di lingkungan lokal menggunakan **Minikube** pada sistem berbasis Linux (Ubuntu).

### Prasyarat
Pastikan perangkat lunak berikut sudah terpasang:
*   [Java JDK 21+](https://www.oracle.com/java/technologies/downloads/)
*   [Maven 3.x](https://maven.apache.org/download.cgi)
*   [Docker](https://docs.docker.com/engine/install/ubuntu/)
*   [Minikube](https://minikube.sigs.k8s.io/docs/start/)
*   [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

### Langkah 1: Clone dan Build Proyek

1.  **Clone repositori ini dan masuk ke direktorinya.**
    ```bash
    git clone https://github.com/alrifqidarmawan/technical-assessment.git
    cd technical-assessment
    ```

2.  **Build aplikasi menjadi file `.jar` menggunakan Maven.**
    ```bash
    mvn clean package
    ```

3.  **Build image Docker dari Dockerfile.**
    *Ganti `alrifqidarmawan` dengan username Docker Hub Anda jika perlu.*
    ```bash
    docker build -t alrifqidarmawan/app-technical-assessment:latest .
    ```
    > **Catatan:** Anda juga dapat mengunggah image ini ke Docker Hub dengan `docker push alrifqidarmawan/app-technical-assessment:latest` agar Kubernetes dapat menariknya dari registry publik.

### Langkah 2: Siapkan Cluster Kubernetes Lokal

1.  **Start cluster Minikube.**
    ```bash
    minikube start --driver=docker
    ```

2.  **Aktifkan Ingress Controller.**
    *Ini hanya perlu dilakukan sekali per cluster.*
    ```bash
    minikube addons enable ingress
    ```

### Langkah 3: Deploy Aplikasi ke Kubernetes

1.  **Terapkan semua manifest yang ada di folder `k8s/`.**
    ```bash
    kubectl apply -f k8s/
    ```

2.  **Tunggu beberapa saat hingga semua Pods berjalan.** Anda bisa memantaunya dengan:
    ```bash
    kubectl get pods -w
    ```
    Tekan `Ctrl+C` setelah kolom `STATUS` untuk semua pod `app-technical-assessment...` menunjukkan `Running`.

### Langkah 4: Akses Aplikasi

1.  **Dapatkan IP dari cluster Minikube Anda.**
    ```bash
    minikube ip
    ```
    Catat alamat IP yang muncul (misalnya: `192.168.49.2`).

2.  **Edit file `/etc/hosts` untuk memetakan domain ke IP Minikube.**
    ```bash
    sudo nano /etc/hosts
    ```
    Tambahkan baris berikut di bagian bawah file, ganti IP-nya dengan yang Anda dapatkan di atas.
    ```
    192.168.49.2    technical-assessment-app.alrifqidarmawan.com
    ```
    Simpan file (`Ctrl+X`, lalu `Y`, lalu `Enter`).

3.  **Buka browser Anda dan kunjungi:**
    > **http://technical-assessment-app.alrifqidarmawan.com**

Anda seharusnya akan disambut dengan halaman dari aplikasi Spring Boot Anda!

---

### 🌐 Endpoint API

Setelah aplikasi berjalan, Anda bisa menguji endpoint menggunakan `curl`.

```bash
curl http://technical-assessment-app.alrifqidarmawan.com/
```
Respons yang diharapkan:
```
Welcome to Technical Assessment!
```

---

### 🧹 Membersihkan Environment

Setelah selesai, gunakan perintah berikut untuk mematikan atau menghapus cluster lokal Anda.

```bash
# Untuk mematikan cluster sementara (bisa di-start lagi)
minikube stop

# Untuk menghapus cluster secara permanen (membebaskan semua resource)
minikube delete
```
> **Penting:** Jangan lupa untuk menghapus baris yang Anda tambahkan di file `/etc/hosts` untuk menghindari masalah di kemudian hari.

Berikut ini adalah template untuk melakukan deploy aplikasi di Kubernetes dengan menggunakan GitHub Actions sebagai workflow CI/CD, serta contoh workflow file (`.github/workflows/deploy.yml`) untuk setiap project (Express.js, Laravel, PHP Native, dan CodeIgniter).

---

# Deploy Aplikasi di Kubernetes dengan GitHub Actions

Panduan ini menjelaskan cara melakukan deploy aplikasi yang dibangun dengan Express.js, Laravel, PHP Native, dan CodeIgniter ke Kubernetes menggunakan GitHub Actions.

## Prasyarat

1. **Kubernetes Cluster** - Pastikan Anda memiliki akses ke cluster Kubernetes.
2. **Docker Hub Account** - Untuk menyimpan Docker images.
3. **GitHub Repository** - Tempat kode aplikasi dan workflow.
4. **GitHub Secrets** - Simpan kredensial seperti `DOCKER_USERNAME`, `DOCKER_PASSWORD`, dan `KUBECONFIG` di Secrets.

## Struktur Repositori

Pastikan repositori Anda memiliki struktur seperti berikut:

```
.
├── app/                    # Source code aplikasi
├── k8s/                    # YAML files untuk Kubernetes manifests
│   ├── deployment.yaml     # Definisi Deployment Kubernetes
│   ├── service.yaml        # Definisi Service Kubernetes
│   └── ingress.yaml        # Definisi Ingress (opsional)
├── .github/                # GitHub Actions workflows
│   └── workflows/
│       └── deploy.yml      # Workflow file untuk CI/CD
├── Dockerfile              # Definisi Docker untuk aplikasi
└── README.md               # Panduan ini
```

## GitHub Actions Workflow

### 1. Setup Secrets di GitHub

Tambahkan secrets di repository GitHub Anda:

- `DOCKER_USERNAME`: Username Docker Hub.
- `DOCKER_PASSWORD`: Password Docker Hub.
- `KUBECONFIG`: File kubeconfig yang dibutuhkan untuk akses cluster Kubernetes.

### 2. Workflow File

Buat file `deploy.yml` di `.github/workflows/` dengan isi sesuai dengan aplikasi yang digunakan.

#### **Express.js Workflow**

```yaml
name: Deploy Express.js App

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/express-app:latest .
        docker push ${{ secrets.DOCKER_USERNAME }}/express-app:latest

    - name: Deploy to Kubernetes
      run: |
        echo "${{ secrets.KUBECONFIG }}" > kubeconfig
        kubectl --kubeconfig=kubeconfig apply -f k8s/deployment.yaml
        kubectl --kubeconfig=kubeconfig apply -f k8s/service.yaml
        kubectl --kubeconfig=kubeconfig apply -f k8s/ingress.yaml
```

#### **Laravel Workflow**

```yaml
name: Deploy Laravel App

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/laravel-app:latest .
        docker push ${{ secrets.DOCKER_USERNAME }}/laravel-app:latest

    - name: Deploy to Kubernetes
      run: |
        echo "${{ secrets.KUBECONFIG }}" > kubeconfig
        kubectl --kubeconfig=kubeconfig apply -f k8s/deployment.yaml
        kubectl --kubeconfig=kubeconfig apply -f k8s/service.yaml
        kubectl --kubeconfig=kubeconfig apply -f k8s/ingress.yaml
```

#### **PHP Native Workflow**

```yaml
name: Deploy PHP Native App

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/php-native-app:latest .
        docker push ${{ secrets.DOCKER_USERNAME }}/php-native-app:latest

    - name: Deploy to Kubernetes
      run: |
        echo "${{ secrets.KUBECONFIG }}" > kubeconfig
        kubectl --kubeconfig=kubeconfig apply -f k8s/deployment.yaml
        kubectl --kubeconfig=kubeconfig apply -f k8s/service.yaml
        kubectl --kubeconfig=kubeconfig apply -f k8s/ingress.yaml
```

#### **CodeIgniter Workflow**

```yaml
name: Deploy CodeIgniter App

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/codeigniter-app:latest .
        docker push ${{ secrets.DOCKER_USERNAME }}/codeigniter-app:latest

    - name: Deploy to Kubernetes
      run: |
        echo "${{ secrets.KUBECONFIG }}" > kubeconfig
        kubectl --kubeconfig=kubeconfig apply -f k8s/deployment.yaml
        kubectl --kubeconfig=kubeconfig apply -f k8s/service.yaml
        kubectl --kubeconfig=kubeconfig apply -f k8s/ingress.yaml
```

## Deploy ke Kubernetes

GitHub Actions akan secara otomatis membangun dan mendepoy aplikasi Anda ke Kubernetes setiap kali ada perubahan yang didorong ke branch `main`.

### 1. Membangun Docker Image dan Deploy

Setiap kali ada commit ke branch `main`, workflow akan:

1. Checkout kode sumber.
2. Membangun Docker image untuk aplikasi.
3. Mendorong Docker image ke Docker Hub.
4. Menerapkan manifest Kubernetes untuk mendepoy aplikasi ke cluster Kubernetes.

### 2. Verifikasi Deployment

Setelah workflow selesai, Anda dapat memeriksa status deployment dengan:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress (opsional)
```

## Kesimpulan

Dengan menggunakan GitHub Actions, Anda telah berhasil mengotomatiskan proses build dan deploy aplikasi Express.js, Laravel, PHP Native, dan CodeIgniter ke Kubernetes. Pastikan untuk menyesuaikan instruksi ini sesuai dengan kebutuhan spesifik aplikasi Anda.

---


---
## Tambahan Berikut merupakan workflow auto backup baik file maupun database

Berikut adalah contoh GitHub Actions workflow yang akan melakukan backup harian terhadap file yang diunggah serta database aplikasi Anda. Setelah backup selesai, workflow ini akan melakukan push backup ke repositori khusus untuk backup.

### Workflow Backup Harian

```yaml
name: Daily Backup

on:
  schedule:
    - cron: '0 2 * * *' # Menjalankan setiap hari pukul 02:00 UTC

jobs:
  backup-files:
    name: Backup Uploaded Files
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Create backup folder
      run: mkdir -p backup/uploads

    - name: Copy uploaded files
      run: |
        cp -r /path/to/your/upload/folder1 backup/uploads/
        cp -r /path/to/your/upload/folder2 backup/uploads/
        # Tambahkan lebih banyak folder sesuai kebutuhan

    - name: Compress backup
      run: tar -czf backup/uploads_backup_$(date +%F).tar.gz -C backup uploads

    - name: Upload to backup repository
      uses: actions/checkout@v3
      with:
        repository: your-username/your-backup-repo
        token: ${{ secrets.GITHUB_TOKEN }}

    - name: Push backup files
      run: |
        git config user.name "GitHub Actions"
        git config user.email "actions@github.com"
        git add backup/uploads_backup_$(date +%F).tar.gz
        git commit -m "Backup files for $(date +%F)"
        git push

  backup-database:
    name: Backup Database
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Backup database
      env:
        DB_USER: ${{ secrets.DB_USER }}
        DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        DB_HOST: ${{ secrets.DB_HOST }}
        DB_NAME: ${{ secrets.DB_NAME }}
      run: |
        mysqldump -u$DB_USER -p$DB_PASSWORD -h$DB_HOST $DB_NAME > backup/db_backup_$(date +%F).sql

    - name: Compress backup
      run: tar -czf backup/db_backup_$(date +%F).tar.gz -C backup db_backup_$(date +%F).sql

    - name: Upload to backup repository
      uses: actions/checkout@v3
      with:
        repository: your-username/your-backup-repo
        token: ${{ secrets.GITHUB_TOKEN }}

    - name: Push backup database
      run: |
        git config user.name "GitHub Actions"
        git config user.email "actions@github.com"
        git add backup/db_backup_$(date +%F).tar.gz
        git commit -m "Backup database for $(date +%F)"
        git push
```

### Penjelasan

1. **schedule**: Workflow ini akan dijalankan setiap hari pada pukul 02:00 UTC.
   
2. **backup-files**: 
   - Membuat direktori backup.
   - Menyalin file yang diunggah dari beberapa folder ke dalam direktori backup.
   - Mengompres file-file tersebut menjadi satu file `.tar.gz`.
   - Mendorong file backup ke repositori khusus.

3. **backup-database**:
   - Melakukan backup database menggunakan `mysqldump`.
   - Mengompres file `.sql` hasil backup menjadi `.tar.gz`.
   - Mendorong file backup ke repositori khusus.

### **Catatan:**
- Ganti `/path/to/your/upload/folder1` dan `/path/to/your/upload/folder2` dengan path ke folder yang berisi file unggahan Anda.
- Pastikan Anda memiliki `secrets` yang diperlukan di GitHub, seperti `DB_USER`, `DB_PASSWORD`, `DB_HOST`, dan `DB_NAME`, serta `GITHUB_TOKEN` yang secara otomatis tersedia di GitHub Actions.
- Ganti `your-username/your-backup-repo` dengan nama repositori backup Anda.

Dengan workflow ini, setiap hari, backup file dan database Anda akan dilakukan secara otomatis, dan hasil backup akan disimpan di repositori khusus untuk keamanan dan kemudahan akses.

---


---
## Tambahan Berikut merupakan workflow auto scan vuln pada dependencies yang perlu di lakukan update

Berikut adalah contoh GitHub Actions workflow yang dapat Anda gunakan untuk melakukan pemindaian kerentanan terhadap paket yang diinstal. Workflow ini menggunakan alat seperti npm audit untuk proyek Node.js (Express.js) dan composer audit untuk proyek PHP (Laravel, PHP Native, CodeIgniter). Workflow ini di eksekusi setiap satu minggu sekali

### Workflow untuk Pemindaian Kerentanan Mingguan

```yaml
name: Weekly Vulnerability Scan

on:
  schedule:
    - cron: '0 0 * * 0' # Menjalankan setiap hari Minggu pukul 00:00 UTC

jobs:
  scan-express:
    name: Scan Express.js Dependencies
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'

    - name: Install dependencies
      run: npm install

    - name: Audit dependencies
      run: npm audit --audit-level=moderate

  scan-laravel:
    name: Scan Laravel Dependencies
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.0'

    - name: Install Composer
      run: composer install

    - name: Audit dependencies
      run: composer audit --format=json

  scan-php-native:
    name: Scan PHP Native Dependencies
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.0'

    - name: Install Composer
      run: composer install

    - name: Audit dependencies
      run: composer audit --format=json

  scan-codeigniter:
    name: Scan CodeIgniter Dependencies
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.0'

    - name: Install Composer
      run: composer install

    - name: Audit dependencies
      run: composer audit --format=json
```

### Penjelasan

- **schedule**: Menggunakan cron untuk menjalankan workflow setiap minggu pada hari Minggu pukul 00:00 UTC (`'0 0 * * 0'`).
- **scan-express, scan-laravel, scan-php-native, scan-codeigniter**: Bagian ini memindai kerentanan pada aplikasi yang berbeda.

Dengan workflow ini, pemindaian kerentanan akan dilakukan secara otomatis setiap minggu, memastikan bahwa Anda terus mendapatkan informasi terbaru tentang potensi masalah keamanan pada proyek Anda.

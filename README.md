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

wicaksu

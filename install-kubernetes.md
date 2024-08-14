### Tutorial Lengkap Instalasi Kubernetes untuk Produksi dengan NGINX Ingress Controller dan Dashboard Kubernetes

Dalam tutorial ini, kita akan melakukan instalasi Kubernetes yang aman dan fleksibel, termasuk pengaturan NGINX Ingress Controller sebagai load balancer, dan instalasi dashboard Kubernetes dengan akses via domain. Tutorial ini cocok untuk lingkungan produksi dan menggunakan node dengan IP publik.

### Prasyarat

- Setiap node (baik master maupun worker) memiliki akses internet dan IP publik.
- Sistem operasi yang digunakan adalah Ubuntu 20.04.
- Akses root atau `sudo` pada setiap node.

### 1. Persiapan Lingkungan

#### 1.1. Update Sistem dan Instalasi Dependensi

Jalankan perintah berikut di semua node (baik master maupun worker):

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

#### 1.2. Matikan Swap Memory

Kubernetes memerlukan swap memory untuk dinonaktifkan. Lakukan perintah berikut di semua node:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

#### 1.3. Instalasi Docker

Instal Docker sebagai container runtime di semua node:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
sudo systemctl enable docker
sudo systemctl start docker
```

### 2. Instalasi Kubernetes

#### 2.1. Instalasi Kubeadm, Kubelet, dan Kubectl

Instal kubeadm, kubelet, dan kubectl di semua node:

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo bash -c 'cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF'
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

#### 2.2. Inisialisasi Master Node

Pada master node, jalankan perintah berikut untuk menginisialisasi cluster Kubernetes:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<IP_PUBLIK_MASTER>
```

Gantilah `<IP_PUBLIK_MASTER>` dengan IP publik master node Anda. Setelah inisialisasi selesai, ikuti petunjuk untuk mengkonfigurasi `kubectl`:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 2.3. Instalasi Network Plugin (Calico)

Instal Calico sebagai network plugin di cluster:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

#### 2.4. Menambahkan Worker Nodes ke Cluster

Pada setiap worker node, jalankan perintah berikut untuk bergabung ke cluster:

```bash
sudo kubeadm join <IP_PUBLIK_MASTER>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

Gantilah `<IP_PUBLIK_MASTER>`, `<TOKEN>`, dan `<HASH>` dengan informasi yang diberikan oleh perintah `kubeadm init` pada master node.

### 3. Instalasi NGINX Ingress Controller

#### 3.1. Instalasi Helm

Jika Helm belum terinstal, instal dengan perintah berikut:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

#### 3.2. Instalasi NGINX Ingress Controller

Tambahkan repository Helm untuk NGINX Ingress Controller:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

Instal NGINX Ingress Controller:

```bash
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --set controller.service.type=LoadBalancer \
    --set controller.service.externalTrafficPolicy=Local
```

#### 3.3. Verifikasi Instalasi

Cek apakah NGINX Ingress Controller telah berhasil terinstal dengan perintah berikut:

```bash
kubectl get svc -o wide -w -n default
```

Cari service `nginx-ingress-ingress-nginx-controller` yang akan memiliki LoadBalancer IP yang bisa digunakan untuk mengakses aplikasi.

### 4. Setup Ingress untuk Aplikasi

#### 4.1. Buat Resource Ingress

Buat file YAML untuk Ingress. Misalnya:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: your-service
            port:
              number: 80
```

Ganti `yourdomain.com` dengan domain atau subdomain Anda dan `your-service` dengan nama service yang ingin diakses.

Terapkan konfigurasi Ingress ini:

```bash
kubectl apply -f example-ingress.yaml
```

#### 4.2. Konfigurasi DNS

Arahkan domain atau subdomain Anda ke LoadBalancer IP yang diberikan oleh NGINX Ingress Controller.

### 5. Instalasi Dashboard Kubernetes

#### 5.1. Instalasi Dashboard

Instal dashboard Kubernetes dengan perintah berikut:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
```

#### 5.2. Buat Service Account dan Role Binding

Untuk mengakses dashboard dengan hak akses admin, buat service account dan role binding:

```bash
kubectl create serviceaccount dashboard-admin-sa
kubectl create clusterrolebinding dashboard-admin-sa --clusterrole=cluster-admin --serviceaccount=default:dashboard-admin-sa
```

#### 5.3. Dapatkan Token untuk Login

Dapatkan token untuk login ke dashboard:

```bash
kubectl get secret $(kubectl get serviceaccount dashboard-admin-sa -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
```

Simpan token ini untuk login ke dashboard.

#### 5.4. Setup Akses Dashboard via Domain

Jika Anda ingin mengakses dashboard melalui domain, buat Ingress resource khusus untuk dashboard:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: dashboard.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 80
```

Terapkan konfigurasi ini:

```bash
kubectl apply -f dashboard-ingress.yaml
```

Pastikan domain `dashboard.yourdomain.com` diarahkan ke LoadBalancer IP.

### 6. Setup HTTPS dengan Cert-Manager (Opsional)

#### 6.1. Instalasi Cert-Manager

Tambahkan repo cert-manager ke Helm:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

Instal cert-manager:

```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.9.1/cert-manager.yaml
```

#### 6.2. Buat ClusterIssuer untuk Let's Encrypt

Buat file `cluster-issuer.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

Terapkan ClusterIssuer:

```bash
kubectl apply -f cluster-issuer.yaml
```

#### 6.3. Konfigurasi Ingress untuk Mendapatkan Sertifikat SSL/TLS

Update file Ingress untuk aplikasi dengan anotasi cert-manager:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - yourdomain.com
    secretName: yourdomain-tls
  rules:
  - host: yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
       

 backend:
          service:
            name: your-service
            port:
              number: 80
```

Terapkan konfigurasi ini:

```bash
kubectl apply -f example-ingress.yaml
```

Cert-manager akan mengelola sertifikat SSL/TLS dari Let's Encrypt, dan NGINX Ingress Controller akan menggunakan sertifikat tersebut.

### 7. Akses Dashboard Kubernetes

Sekarang, Anda dapat mengakses dashboard Kubernetes melalui domain yang telah Anda setup (`dashboard.yourdomain.com`). Login menggunakan token yang Anda dapatkan sebelumnya.

### Penutup

Dengan mengikuti langkah-langkah ini, Anda telah berhasil menginstal Kubernetes yang aman dan fleksibel di lingkungan produksi, dengan load balancing menggunakan NGINX Ingress Controller, dan mengakses dashboard Kubernetes melalui domain yang aman. Jangan lupa untuk terus memantau dan mengamankan cluster Kubernetes Anda sesuai dengan kebutuhan produksi.

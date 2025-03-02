---
title: "Implementing a Monitoring Solution for Kubernetes Cluster and Resource Usage"
author: Amanda 
date: 2025-01-21 08:46
tag:
- Cluster
- Kubernetes
- Monitoring
category: [Blogging, Tutorial]
---

Heyyhoo, teman-teman! 👋
Selamat datang di blog ini! Di sini, saya bakal berbagi pengalaman tentang cara mengimplementasikan solusi monitoring untuk cluster Kubernetes dan penggunaan sumber daya. Kita akan kupas tuntas langkah-langkahnya supaya sistem tetap sehat, efisien, dan berjalan optimal. Yuk, kita eksplor bareng!

## Deskripsi
Perusahaan telah mengoperasikan cluster Kubernetes yang mendukung berbagai aplikasi
yang berjalan bersamaan. Dengan semakin banyaknya aplikasi yang dikelola, memastikan bahwa
cluster Kubernetes berjalan dengan lancar dan efisien menjadi sangat penting. Untuk itu, manajer
tim operasional mengidentifikasi kebutuhan untuk memantau secara real-time status cluster, status
pod, dan penggunaan sumber daya pada cluster Kubernetes yang ada.

Manajer tim operasional memutuskan untuk mengembangkan sistem monitoring yang
dapat memberikan visibilitas menyeluruh terhadap kondisi cluster, termasuk deteksi potensi
masalah yang dapat mempengaruhi kinerja aplikasi. Dengan sistem monitoring ini, tim dapat
memantau penggunaan sumber daya seperti CPU, memori, dan storage, serta memastikan bahwa
setiap pod yang berjalan di dalam cluster tetap stabil dan berfungsi dengan baik.

Sistem monitoring ini akan membantu tim operasional dalam mengambil keputusan yang
cepat dan tepat, mengurangi potensi downtime, serta meningkatkan efisiensi dan keandalan
cluster Kubernetes secara keseluruhan. Dengan demikian, implementasi monitoring yang efektif
menjadi langkah strategis dalam menjaga performa dan ketersediaan aplikasi yang berjalan di
dalam cluster.

## Tools yang DIgunakan:
- Kubernetes v1.28.15
- Containerd
- Kubeadm
- Kubectl
- Kubelet
- Helm v3
- Prometheus v3.0.1
- Grafana v11.4.0

## Topologi
![topologi](/assets/img/git%20clone%20(3).png)

## Workflow
![alur kerja](/assets/img/git%20clone%20(2).png)

## Teori
### 1. Cluster
Cluster adalah sekumpulan komputer atau server yang terhubung dalam sebuah jaringan
dan bekerja bersama untuk menjalankan aplikasi, layanan, atau tugas tertentu. Setiap komputer
dalam cluster disebut node, dan mereka saling berkomunikasi untuk berbagi beban kerja atau
memastikan ketersediaan layanan. Cluster digunakan untuk meningkatkan kinerja, skalabilitas,
dan keandalan sistem.

### 2. Metrics 
Metrics adalah data atau informasi yang digunakan untuk mengukur performa atau
keadaan suatu sistem. Dalam monitoring, metrics bisa berupa data seperti penggunaan CPU,
memori, jumlah permintaan yang diterima server, atau waktu respon aplikasi. Metrics membantu
untuk memantau dan menganalisis kinerja sistem secara terus-menerus, sehingga bisa
mengetahui apakah sistem berfungsi dengan baik atau perlu diperbaiki.

### 3. Alert 
Alert adalah notifikasi atau peringatan yang dikirimkan ketika sistem mendeteksi adanya
masalah, anomali, atau kondisi tertentu yang memerlukan perhatian. Alert biasanya dihasilkan oleh
alat monitoring seperti Prometheus atau sistem lainnya berdasarkan aturan atau rules yang sudah
ditentukan. Alert bisa dikirim ke berbagai saluran, seperti email, chat (Slack, Microsoft Teams), atau
sistem notifikasi lainnya, sehingga tim dapat segera mengambil tindakan untuk menyelesaikan
masalah sebelum berdampak besar.

### 4. Monitoring 
Monitoring adalah proses memantau kinerja, status, atau kondisi suatu sistem atau aplikasi
secara terus-menerus. Tujuannya adalah untuk memastikan bahwa sistem berjalan dengan baik
dan mendeteksi masalah sebelum menjadi lebih besar. Dalam monitoring, biasanya
mengumpulkan data seperti penggunaan sumber daya (CPU, memori, dll.), kesehatan aplikasi, atau
kecepatan respon, dan kemudian dianalisis untuk mengambil tindakan yang diperlukan jika ada
masalah.

### 5. Kubernetes
Kubernetes adalah platform open-source yang digunakan untuk mengelola dan mengatur
aplikasi yang dikemas dalam container. Dengan Kubernetes, proses penyebaran, penskalaan, dan
pengelolaan aplikasi menjadi lebih otomatis dan efisien. Awalnya dikembangkan oleh Google,
Kubernetes kini dikelola oleh Cloud Native Computing Foundation (CNCF). Dengan menggunakan
Kubernetes, dapat mengelola aplikasi dalam container secara lebih mudah dan efisien, serta
memastikan aplikasi berjalan dengan baik di berbagai lingkungan.

### 6. Helm
Helm adalah sebuah package manager untuk Kubernetes yang digunakan untuk
mengelola aplikasi dalam bentuk “chart”. Chart ini berisi semua file konfigurasi yang diperlukan
untuk mendefinisikan, menginstal, dan mengelola aplikasi di dalam cluster Kubernetes. Dengan
helm, dapat menginstall aplikasi dengan mudah, mengelola versi aplikasi, dan mengatur
konfigurasi aplikasi.

### 7. Containerd
Containerd atau container runtime adalah perangkat lunak yang berfungsi untuk
menjalankan dan mengelola container. Container itu sendiri adalah unit terkecil dari aplikasi yang
sudah dikemas dengan semua dependensinya, sehingga dapat dijalankan di berbagai lingkungan.
Containerd adalah salah satu jenis container runtime yang digunakan dalam sistem seperti
Kubernetes. Containerd berfungsi untuk menjalankan container, mengelola lifecycle container
(mulai, stop, dan hapus), dan menyediakan interface agar container dapat berinteraksi dengan
sistem.

### 8. Prometheus 
Prometheus adalah alat open-source untuk mengumpulkan dan menyimpan data
monitoring dalam bentuk time-series, dirancang untuk memantau aplikasi dan sistem secara
otomatis. Alat ini dapat digunakan untuk memonitor berbagai metrik, seperti penggunaan CPU,
memori, atau status aplikasi, dengan cara menarik (scraping) data dari target yang sudah
ditentukan dan menyimpannya dalam database internal.

### 9.  Grafana
Grafana adalah alat open-source yang digunakan untuk visualisasi data. Biasanya, Grafana
digunakan untuk membuat dashboard yang menampilkan metrik dan data yang dikumpulkan dari
berbagai sumber, seperti Prometheus, Elasticsearch, atau database lainnya. Dengan Grafana,
pengguna dapat membuat grafik, diagram, dan visualisasi lainnya untuk memantau kinerja sistem,
aplikasi, atau infrastruktur secara real-time.

## Langkah Implementasi
### Persiapan Environment
> Eksekusi pada semua node.
{: .prompt-info }
- Petakan host di dalam /etc/hosts.

```bash
$ sudo nano /etc/hosts

192.168.2.10 node-master
192.168.2.20 node-worker01
192.168.2.30 node-worker02
```

- Buat ssh-keygen, lalu copy public key dari pengguna biasa ke semua node.

```bash
$ ssh-keygen -t rsa

$ ssh-copy-id student@node-master
$ ssh-copy-id student@node-worker01
$ ssh-copy-id student@node-worker02
```

- Verifikasi mengakses tanpa kata sandi.

```bash
$ ssh student@node-master "whoami; hostname"
$ ssh student@node-worker01 "whoami; hostname"
$ ssh student@node-worker02 "whoami; hostname"
```

### Install Kubernetes
> Eksekusi pada semua node.
{: .prompt-info }
- Update dan upgrade paket.

```bash
$ sudo apt update
$ sudo apt upgrade -y
$ sudo apt autoremove -y
```

- Install depedensi dan menambahkan repository untuk menginstall containerd.

```bash
$ sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

- Install containerd dan konfigurasi containerd.

```bash
$ sudo apt update
$ sudo apt install -y containerd.io
$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
$ sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

- Restart dan enable service containerd.

```bash
$ sudo systemctl restart containerd
$ sudo systemctl enable containerd
```

- Tambahkan kernel setting.

```bash
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```
> Modul overlay digunakan untuk menyimpan dan mengelola file system layer dari container, sedangkan br_netfilter untuk memproses lalu lintas jaringan, seperti digunakan untuk komunikasi antar-pod di dalam Kubernetes.
{: .prompt-tip }

- Konfigurasi iptables menggunakan sysctl.

```bash
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

$ sudo sysctl --system
```

- Menambahkan repository untuk install kubectl, kubelet dan kubeadm.

```bash
$ sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

> Eksekusi pada node master.
{: .prompt-info } 
- Menonaktifkan swap yang ada dan initialize node master Kubernetes.

```bash
$ swapon -s
$ sudo swapoff -a
$ sudo kubeadm init --pod-network-cidr=10.244.XX.0/16
```

- Salin konfigurasi admin Kubernetes ke direktori pengguna dan mengubah kepemilikan.

```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- Install POD Network Flannel.

```bash
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl apply -f kube-flannel.yml
$ kubectl get pods --all-namespaces --watch
```
> Flannel adalah plugin jaringan yang digunakan untuk mengelola komunikasi antar pod.
{: .prompt-tip }

- Menampilkan token dan token sertifikat CA.

```bash
$ sudo kubeadm token list
$ sudo openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

> Eksekusi pada node worker.
{: .prompt-info }
- Nonaktifkan swap, lalu join node worker ke node master.
 
 ```bash
$ swapon -s
$ sudo swapoff -a
$ sudo kubeadm join --token [TOKEN] [NODE-MASTER]:6443 --discovery-token-ca-cert-hash sha256:[TOKEN-CA-CERT-HASH]
```

> Eksekusi pada node master.
{: .prompt-info }
- Verifikasi node.

```bash
$ kubectl get nodes
```

### Install Helm
> Eksekusi pada node master.
{: .prompt-info }
- Import public key Helm, lalu install depedensi dan tambahkan repositori Helm.

```bash
$ curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
$ sudo apt-get install apt-transport-https --yes
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
```

- Update packages dan install Helm.

```bash
$ sudo apt-get update
$ sudo apt-get install helm
```

### Install Prometheus dan Grafana menggunakan Helm
> Eksekusi pada node master.
{: .prompt-info }
- Buat namespace untuk menyediakan tempat terpisah di cluster Kubernetes untuk Prometheus dan Grafana.

```bash
$ kubectl create namespace monitoring
```

- Tambahkan repositori dari komunitas, lalu update repo nya.

```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
```

- Install Prometheus Helm Chart di Kubernetes cluster.

```bash
$ helm install my-kube-prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring
```
> Install nya sudah termasuk Prometheus dan Grafana.
{: .prompt-info }

- Jalankan command berikut untuk melihat semua yang sudah di install oleh Helm di cluster Kubernetes.
  
```bash
$ kubectl get all -n monitoring
```

- Jalankan command `kubectl get service`{: .filepath} untuk melihat service untuk Prometheus dan Grafana.

```bash
$ kubectl get service -n monitoring
```

- Mengekspos Prometheus dan Grafana menggunakan service NodePort.

```bash
$ kubectl expose service my-kube-prometheus-stack-prometheus --namespace monitoring --type=NodePort --target-port=9090 --name=prometheus-node-port-service
$ kubectl expose service my-kube-prometheus-stack-grafana --namespace monitoring --type=NodePort --target-port=3000 --name=grafana-node-port-service
```

### Akses Prometheus Web
- Akses Prometheus dengan menggunakan IP dengan port yang sudah dibuat sebelumnya. (contoh: 192.168.2.10:32217).
![task1](/assets/img/task1.png)

### Akses Grafana dan Membuat Visualisasi
- Akses web Grafana dengan mengguanakan IP dan port yang sudah di buat sebelumnya. (contoh: 192.168.2.10:30607).
![task2](/assets/img/task2.png)

- Jalankan command dibawah ini, untuk mendapatkan kata sandi pengguna admin.

```bash
$ kubectl get secret --namespace monitoring my-kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

- Tampilan setelah masuk ke dalam dashboard Grafana.
![task3](/assets/img/task3.png)

- Masuk ke dalam menu home lalu pilih `Dashboard.`{: .filepath}
![task4](/assets/img/task4.png)

- Setelah itu, klik `New`{: .filepath} > `New dashboard.`{: .filepath}
![task5](/assets/img/task5.png)

> Secara default Helm akan menambahkan beberapa dashboard untuk memantau Cluster Kubernetes dan resource nya.
{: .prompt-info }

- Lalu klik `+ Add visualization`{: .filepath} untuk membuat visualisasi baru.
![task6](/assets/img/task6.png)

- Masukkan data source.
![task7](/assets/img/task7.png)

- Lalu buat visualisasi, disini dibuat untuk status kondisi node dengan query _kube_node_status_condition_. Isi query > lalu isi `Options Legend`{: .filepath} dengan node > pilih visualisasi `Gauge`{: .filepath} > beri nama untuk panel nya > `Save dashboard.`{: .filepath}
![task8](/assets/img/task8.png)

> Di query ini akan menampilkan kondisi node yang statusnya ready/siap.
{: .prompt-info }

- Setelah semuanya sudah, save dashboard dan beri nama untuk dashboardnya.
![task9](/assets/img/task9.png)

- Query untuk memonitoring penggunaan CPU dalam skala persen. Isi query > pilih visualisasi `Stat`{: .filepath}, ke bagian `Color mode`{: .filepath} lalu pilih yang `Background Gradient`{: .filepath}, ke bagian Standard options pilih unit `Misc Percent (0.0-1.0)`{: .filepath} > beri nama lalu save.
![task10](/assets/img/task10.png)

- Query untuk memonitoring penggunaan memori dalam skala persen. Isi query > lalu pilih visualisasi `Stat`{: .filepath} (dan untuk tampilan seperti ini kurang lebih sama seperti cara sebelumnya)> beri nama lalu save.
![task11](/assets/img/task11.png)

- Query untuk memonitoring total penggunaan CPU per Namespace. Isi query > pilih visualisasi `Time Series`{: .filepath} > beri nama lalu save
![task12](/assets/img/task12.png)

> Query ini menggunakan sum yang artinya menjumlahkan nilai penggunaan CPU dalam setiap namespace
{: .prompt-info }

- Query untuk memonitoring total penggunaan memori RSS (Resident Set Size) setiap container per Namespace. Isi query > pilih visualisasi `Time Series`{: .filepath} > beri nama lalu save.
![task13](/assets/img/task13.png)

> Query ini menggunakan label job=kubelet yang berarti metrik dikumpulkan dari kubelet dan metric_path berarti data ini berasal dari cAdvisor yang digunakan kubelet untuk memantau container
{: .prompt-info }

- Setelah semuanya selesai Save Dashboard.
![task14](/assets/img/task14.png)
![task15](/assets/img/task15.png)

### Konfigurasi Alert Di Grafana
- Untuk membuat alert di Grafana beralih ke bagian `Alerting`{: .filepath} lalu pilih `Contact Points`{: .filepath}, lalu pilih `Create contact point.`{: .filepath}
![task16](/assets/img/task16.png)

- Isi dibagian `Integration`{: .filepath} dengan Discord dan `Webhook URL`{: .filepath} dengan webhook dari Discord, setelah itu beri nama.
![task17](/assets/img/task17.png)

> Untuk mengambil webhook Discord bisa dilihat di bagian referensi.
{: .prompt-info }

- Test mengirim alert ke Discord.
![task18](/assets/img/task18.png)

- Tampilan ketika ada alert yang dikirim oleh Grafana.
![task19](/assets/img/task19.png)

> Untuk alert sudah ada beberapa template yang tersedia ketika install
{: .prompt-info }


## Referensi
[Install Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

[Install Helm](https://helm.sh/docs/intro/install/)

[Prometheus and Grafana on Kubernetes using Helm](https://www.bigbinary.com/blog/prometheus-and-grafana-integration)

[Monitoring a Kubernetes cluster using Prometheus and Grafana](https://medium.com/@akilblanchard09/monitoring-a-kubernetes-cluster-using-prometheus-and-grafana-8e0f21805ea9)

[Common metrics of kube-state-metrics](https://docs.byteplus.com/id/docs/vmp/Common-metrics-of-kube-state-metrics)

[Making a Webhook](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)

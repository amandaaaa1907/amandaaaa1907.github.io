---
title: "Implementing DevSecOps using GitLab SAST in CI Pipeline"
author: Amanda 
date: 2024-11-22 15:17
tag:
- DevSecOps
- GitLab
- SAST
category: [Blogging, Tutorial]
---

Heyyhoo, temen temen semuanya👋👋 Jadi di Blog ini merupakan dokumentasi tentang cara menerapkan konsep `DevSecOps`{: .filepath} dengan memanfaatkan GitLab SAST (Static Application Security Testing) di CI Pipeline. Di sini, akan membahas langkah-langkah untuk mengintegrasikan keamanan aplikasi ke dalam pengembangan software.

Udah penasaran???? Yukk langsung aja!!

## Deskripsi
<p>Dalam Era digitalisasi yang terus berkembang, keamanan perangkat lunak menjadi aspek
yang sangat penting bagi perusahaan. Untuk itu, tim DevOps bertanggung jawab untuk mengelola
semua repositori dengan efisien dan keamanan yang tinggi. Manajer tim DevOps mengambil
langkah strategis dengan mengimplementasikan pendekatan DevSecOps pada salah satu
repositori yang dikelola.</p>
<p>Implementasi DevSecOps ini dilakukan dengan memanfaatkan GitLab SAST (Static
Application Security Testing) di dalam CI (Continuous Integration) Pipeline. Dengan menggunakan
GitLab SAST, setiap perubahaan atau push yang dilakukan pada repositori secara otomatis
menjalani proses pemindaian untuk mendeteksi potensi kerentanan (vulnerbilities) dalam kode.</p>
<p>Hal ini memungkinkan tim untuk mengidentifikasi dan mengatasi masalah keamanan sejak
tahap awal dalam siklus pengembangan, sehingga meningkatkan kualitas dan keamanan
perangkat lunak yang dihasilkan. Ini adalah langkah penting dalam menciptakan budaya
keamanan yang berkelanjutan di dalam tim pengembangan perangkat lunak.</p>

## Tools yang digunakan:
- GitLab Community Edition v16.8.1
- GitLab Runner v17.5.2
- SAST
- Docker v27.3.1
- Python Alpine

## Topologi
![Topologi](/assets/img/git-clone.png)

## Alur Kerja
![Alur Kerja](/assets/img/alur-kerja.png)

<p>Nah, sebelum ke langkah pengerjaan, kita bahas teori nya dulu yukk!</p>

## Teori 
### 1. DevOps
DevOps adalah pendekatan yang menggabungkan tim _Development_ (pengembangan) dan _Operations_ (operasional) supaya bisa kerja sama lebih efektif. Dengan DevOps, proses build, tes, dan deploy aplikasi jadi lebih cepat, efisien, dan minim kesalahan.

### 2. DevSecOps
DevSecOps adalah pengembangan dari DevOps dengan tambahan fokus keamanan. Tes keamanan dilakukan di setiap tahap, bukan hanya di akhir. Tujuannya biar aplikasi nggak hanya selesai cepat, tetapi juga aman dari risiko celah keamanan _(vulnerabilities)_.

### 3. GitLab
GitLab adalah platform lengkap untuk manajemen proyek dan pengembangan aplikasi. Mulai dari _plan_, menyimpan kode, ngejalanin tes, sampai _deploy_, semuanya bisa dilakukan di GitLab. Tim juga bisa kerja bareng lebih mudah karena semuanya ada dalam satu tempat.

### 4. GitLab Runner
Gitlab Runner itu alat buat ngejalanin tugas di pipeline CI/CD GitLab, seperti _build_, tes, atau _deploy_ kode. Runner fleksibel, bisa jalan di berbagi lingkungan, seperti Docker, shell, atau Kubernetes.

### 5. SAST
GitLab SAST _(Static Application Security Testing)_ adalah fitur di GitLab yang otomatis ngecek kode aplikasi buat nyari celah keamanan. Analisisnya dilakukan sebelum aplikasi dijalankan. Hasilnya disimpan dalam laporan berformat JSON.

### 6. Pipeline
Pipeline di GitLab adalah rangkaian tugas otomatis yang jalan secara berurutan, dari _build_, tes, sampai _deploy_. Pipeline bikin proses pengembangan jadi lebih cepat dan konsisten.

### 7. Container
Container adalah unit standar yang membungkus aplikasi dan semua dependensinya. Dengan container, aplikasi bisa jalan di berbagai lingkungan (laptop, server, cloud) tanpa masalah.

### 8. Docker
Docker adalah alat buat bikin dan ngejalanin container. Dengan Docker, kita bisa nge-_package_ aplikasi dan depedensinya jadi satu, sehingga mudah dijalankan di mana saja dengan cara yang sama.

### 9. Python Flask
Flask adalah framework web berbasis Python yang simpel dan ringan. Framework ini cocok buat bikin aplikasi web sederhana atau _prototype_ karena nggak terlalu banyak aturan dan fleksibel kalau mau dikembangkan lebih lanjut.

## Langkah Implementasi
### Install Docker
```bash
# Add Docker's official GPG key:
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Install GitLab Runner
- Install Gitlab Runner lalu tambahkan mode execute.
```bash
$ sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
$ sudo chmod +x /usr/local/bin/gitlab-runner
```

- Buat pengguna untuk Gitlab CI.
```bash
$ sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
```

- Install dan jalankan service Gitlab Runner.
```bash
$ sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
$ sudo gitlab-runner start
```

- Tambahkan pengguna Gitlab Runner ke grup docker.
```bash
$ sudo usermod -aG docker gitlab-runner
$ sudo -u gitlab-runner -H docker info
```

### Membuat GitLab Akun dan Project
- Buat akun Gitlab terlebih dahulu, login atau bisa juga register.
   ![GitLab](/assets/img/ss9.png)
- Untuk membuat project baru pilih `Projects`{: .filepath} > `New Project`{: .filepath} lalu klik `"Create blank project".`{: .filepath}
   ![GitLab](/assets/img/ssganti1.png)
- Isi nama project, pilih Visibility Level `Public`{: .filepath}, lalu klik `"Create project".`{: .filepath}
   ![GitLab](/assets/img/ssganti2.png)

### Clone Repository dari Github dan Konfigurasi GitLab
- Clone repository dari github <https://github.com/bta-adinusa/flask-todo-app>
- Masuk ke dalam direktori dari repository tersebut.
- Buat konfigurasi global untuk menyediakan informasi tentang nama dan email.
```bash
$ git config --global user.name "masukkan username"
$ git config --global user.mail "masukkan email"
$ git config --global init.defaultBranch main
$ git config --global --list
```
- Ubah URL repository menjadi URL ke repository Gitlab, setelah itu verifikasi hasil set url.
```bash
$ git remote set-url origin https://gitlab.com/username/repo.git
$ git remote -v
```

### Membuat Dockerfile
> Karena mendeploy web flask-todo-app nya meggunakan docker jadi buat terlebih dahulu Dockerfile nya.
{: .prompt-info }
Buat Dockerfile nya. Lalu isi Dockerfile sesuai dengan kebutuhan yang diperlukan, seperti contoh dibawah ini.
```bash
$ nano Dockerfile

FROM python:alpine
RUN mkdir /app
RUN apk update

COPY . /app

WORKDIR /app

RUN python3 -m venv /app/venv
RUN /app/venv/bin/pip
RUN pip install -r requirements.txt

ENTRYPOINT ["python"]
CMD ["app.py"]
```
> Karena menggunakan Alpine Linux, gunakan perintah apk update untuk memperbarui package list.
{: .prompt-info }

### Membuat file .gitlab-ci.yml dan SAST.gitlab-ci.yml
- Buat file untuk gitlab ci. Lalu isi file gitlab-ci nya sesuai dengan kebutuhan, seperti contoh dibawah ini.
  
```yaml
$ nano .gitlab-ci.yml

include:
  - template: Security/SAST.gitlab-ci.yml

stages:
  - sast
  - deploy

sast:
  stage: sast
  script:
    - echo "Running SAST"

deploy:
  stage:
  image: docker:latest # gunakan image docker
  needs:
    - job: sast
      artifacts: true
  script:
    - docker build -t (nama untuk image) .
    - docker run -d -p 5000:5000 --name (nama container) (nama image)
    - docker images
    - docker ps
    - sleep 30
    - docker ps
  only:
    - main
```
- Buat file untuk konfigurasi SAST. Karena menggunakan template untuk konfigurasi SAST nya, teman-teman bisa gunakan template yang sudah di sediakan Gitlab.

### Membuat GitLab Runner dan Push Kode
- Sebelum membuat runner, ambil token terlebih dahulu di bagian `Settings`{: .filepath} > `CI/CD.`{: .filepath}
    ![GitLab](/assets/img/ssganti3.png)

- Klik `Expand`{: .filepath} pada bagian Runners.
   ![GitLab](/assets/img/ssganti4.png)

- Klik titik 3 pada Project runners, lalu salin token yang ada.
   ![GitLab](/assets/img/ssganti5.png)

- Register Gitlab Runner dengan menggunakan executor docker. 
```bash
$ sudo gitlab-runner register -n \
  --url https://gitlab.com \
  --registration-token (token) \
  --executor docker \
  --description "Docker Runner" \
  --docker-image "docker:latest" \ 
  --docker-volume /var/run/docker.sock:/var/run/docker.sock
```

- Setelah Runner sudah terbuat, push kode ke gitlab.
```bash
$ git add .
$ git commit -m "comment" && git push origin main
```

### Mendownload Artifacts File JSON
- Untuk mendownload hasil dari artifacts bisa di download di bagian `Build`{: .filepath} > `Artifacts.`{: .filepath} Download file yang format JSON.
   ![GitLab](/assets/img/ssganti6.png)

- Tampilan JSON yang sudah di download akan berantakan dan jadi tidak terstruktur, teman-teman bisa gunakan web untuk memformat ulang tampilan JSON agar jadi lebih terstruktur.
    ![GitLab](/assets/img/ss17.png)

### Membuat Alert Discord Untuk Pipeline dan Vulnerability
> Alert nya disini menggunakan Discord ya.
{: .prompt-info }

- Buka Discord channel tempat yang ingin dipakai untuk menerima notifikasi, lalu pilih `Edit Channel.`{: .filepath}
   ![GitLab](/assets/img/ssganti7.png)

- Pilih bagian `Integrations.`{: .filepath}
   ![GitLab](/assets/img/ssganti8.png)

- Jika belum ada webhook, pilih bagian `Create Webhook`{: .filepath}, lalu pilih `View Webhook`{: .filepath}, setelah itu klik `New Webhook`{: .filepath}. Kalau webhook sudah ada salin URL dari kolom `Copy Webhook URL`{: .filepath}.
   ![GitLab](/assets/img/ssganti9.png)
   ![GitLab](/assets/img/ssganti10.png)

- Beralih ke Gitlab, di bagian `Settings`{: .filepath} pilih Integrations. Pilih `Discord Notifications`{: .filepath}, lalu pilih Configure.
   ![GitLab](/assets/img/ss16.png)

- Di tahap ini masukkan URL webhook yang telah di salin sebelumnya, lalu pilih kotak yang ingin di centang sesuai dengan yang ingin dikirim notifikasi ke Discord. Setelah itu pilih `Save changes`{: .filepath}.
   ![GitLab](/assets/img/ssganti11.png)

- Contoh tampilan notifikasi Discord.
   ![GitLab](/assets/img/ss4.png)
    > Untuk alerting vulnerability menggunakan telegram.
    {: .prompt-info }

- Tambahkan job untuk alert telegram di dalam file _.gitlab-ci.yml_.

```yaml
notify-telegram:
  stage: notify
  image: python:3.9-alpine
  before_script:
    - pip install requests
  script:
    - echo "Sending SAST results to Telegram..."
    - if [ ! -f gl-sast-report,json ]; then echo "Artifact not found!" && exit 1; fi
    - python notify-telegram.py gl-sast-report.json
```

- Edit file _notify-telegram.py_ untuk membuat script alert. 

```bash
$ nano notify-telegram.py

# isi dari file notify-telegram.py
import json
import requests
import sys
import os

# Telegram Bot Token dan Chat ID
TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN")  # TELEGRAM_TOKEN ada di variables gitlab
CHAT_ID = os.getenv("CHAT_ID")  # CHAT_ID ada di variables gitlab

def send_telegram_message(message):
    url = f"https://api.telegram.org/bot{TELEGRAM_TOKEN}/sendMessage"
    payload = {
        "chat_id": CHAT_ID,
        "text": message,
        "parse_mode": "HTML"
    }
    response = requests.post(url, json=payload, timeout=10)
    if response.status_code == 200:
        print("Message sent successfully!")
    else:
        print(f"Failed to send message: {response.status_code}, {response.text}")

def parse_sast_report(json_file):
    with open(json_file, "r") as f:
        data = json.load(f)

    vulnerabilities = data.get("vulnerabilities", [])
    if not vulnerabilities:
        return "✅ <b>No vulnerabilities found</b> in the SAST report."

    # Buat ringkasan hasil
    message = "⚠️ <b>SAST Vulnerabilities Found:</b>\n\n"
    for vuln in vulnerabilities[:5]:  # Kirim maksimal 5 issue
        message += (
            f"🔹 <b>ID:</b> {vuln.get('id', 'N/A')}\n"
            f"    <b>Severity:</b> {vuln.get('severity', 'N/A')}\n"
            f"    <b>Message:</b> {vuln.get('message', 'N/A')}\n"
            f"    <b>File:</b> {vuln.get('location', {}).get('file', 'N/A')} (line {vuln.get('location', {}).get('start_line', 'N/A')})\n\n"
        )
    message += "\n🚨 Check details in the full SAST report."

    return message

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python notify_telegram.py <sast_json>")
        sys.exit(1)

    json_file = sys.argv[1]
    try:
        sast_message = parse_sast_report(json_file)
        send_telegram_message(sast_message)
    except Exception as e:
        print(f"Error: {e}")
        sys.exit(1)
```
- Buat bot di telegram, bot ini untuk mengirimkan alert dari vulnerability.
    ![GitLab](/assets/img/ss5.png)
    > karena saya sudah pernah buat bot sebelumnya, jadi saya rename untuk nama botnya. Setelah bot dibuat salin tokennya.
    {: .prompt-info }
- Akses web untuk mendapatkan informasi bot.
```bash
https://api.telegram.org/bot(token)/getMe
```
- Coba kirim pesan ke bot, setelah itu cek di browser apakah sudah mendapatkan chat id.
    ![GitLab](/assets/img/ss6.png)

- Setelah mendapatkan chat id, masukkan ke dalam variables di gitlab CI/CD. Ke `Settings`{: .filepath} > `CI/CD`{: .filepath} > `Variables`{: .filepath}. Isi variables dengan CHAT_ID dan TELEGRAM_TOKEN.
    ![GitLab](/assets/img/ss7.png)
    ![GitLab](/assets/img/ss8.png)

Nah jadi gitu teman-teman prosesnya, gimana? cukup rumit dan bikin pusing? Dengan ini kita dapat mendeteksi kerentanan sejak dini! Semoga panduan ini memberikan wawasan dan bermanfaat dalam mengimplementasikan GitLab SAST.

Terima kasih telah membaca, dan sampai jumpa di artikel lainnya!👋

## Referensi
[Template SAST](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Jobs/SAST.gitlab-ci.yml)

[GitLab: DevSecOps: Part 5/12: Protect your Apps with Static Application Security Testing (SAST)](https://youtu.be/owwIMUamdDc?si=0qhySUZ5FBd9Lrid)

[Build full CICD Pipelines for Docker Flask App in GitLab](https://youtu.be/jfP11CHsz3E?si=yL9Vbc1Ycf3xkHZY)

[Referensi Website untuk memformat ulang file JSON](https://jsonlint.com/)

[Gunakan BotFather untuk membuar bot](https://web.telegram.org/k/#@BotFather)

[Link Repository](https://github.com/bta-adinusa/flask-todo-app)

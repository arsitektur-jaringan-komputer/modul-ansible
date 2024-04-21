# Deployment With Ansible
Setelah mempelajari materi mengenai ansible pada modul sebelumnya, maka pada modul ini kita akan mengimplementasikan bagaimana cara melakukan deployment dengan Ansible pada case tertentu.

## Table of Contents

1. [Deployment](#deployment)
      - [Langkah 1: Persiapkan Environment](#langkah-1-persiapkan-environment)
      - [Langkah 2: Persiapkan IP VM pada Ansible](#langkah-2-persiapkan-ip-vm-pada-ansible)
      - [Langkah 3: Membuat Playbook](#langkah-3-membuat-playbook)
      - [Langkah 4: Membuat Role](#langkah-4-membuat-role)
      - [Langkah 5: Mengisi task/ Pada Role](#langkah-5-mengisi-task-pada-role)
      - [Langkah 6: Jalankan Ansible](#langkah-6-jalankan-ansible)


# Deployment
Pada modul ini, kita akan mencoba melakukan deployment sederhana dengan menggunakan ansible.

## Langkah 1: Persiapkan Environment
- Persiapkan host target (server) yang akan digunakan untuk deployment.
- Pastikan server telah terkoneksi dengan internet.

## Langkah 2: Persiapkan IP VM pada Ansible
Pada deployment ini, kita akan membuat `inventory.ini` yang akan menyimpan IP dari VM yang digunakan.
```bash
touch inventory.ini
```

Di dalam `inventory.ini`, kita akan memasukkan ip dari vm yang digunakan.
```ini
[web_servers]
<ip_vm>
```
Dari code di atas, dapat diketahui bahwa `web_servers` adalah nama dari group host.

## Langkah 3: Membuat Playbook
Pada deployment ini, kita akan membuat `playbook.yml` yang akan digunakan untuk menavigasi deployment.
```bash
touch playbook.yml
```

Di dalam `playbook.yml`, kita akan memasukkan:
```yml
---
- name: Deploy Simple Website with Nginx
  hosts: web_servers
  become: true
  roles:
    - nginx
```
Dari code ini, dapat diketahui bahwa:
1. name: Ini adalah nama untuk tugas yang akan dilakukan. Dalam hal ini, tugas yang dilakukan adalah "Deploy Simple Website with Nginx".
2. hosts: Menentukan target dari playbook. Dalam hal ini, playbook akan dijalankan pada host-host yang terdaftar di dalam grup `web_servers` di inventory.
3. become: Ini adalah parameter yang menginstruksikan Ansible untuk menjalankan perintah sebagai superuser (biasanya root). Ini diperlukan untuk melakukan tindakan yang memerlukan izin superuser, seperti instalasi paket atau pengaturan konfigurasi sistem.
4. roles: Ini adalah daftar peran (roles) yang akan dieksekusi oleh playbook. Di sini, kita hanya memiliki satu peran yaitu nginx, yang akan menangani instalasi dan konfigurasi `Nginx`.


## Langkah 4: Membuat Role
Dalam case ini, kita hanya akan membutuhkan 1 role, sehingga command yang akan dijalankan adalah sebagai berikut:
```bash
ansible-galaxy init nginx
```
Di sini kita akan menggunakan nama role berupa `nginx`.


## Langkah 5: Mengisi task/ Pada Role
Masukkan file html `index.html` berikut ke dalam `template/`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple WebApp</title>
</head>
<body>
    <h1>Hello, world!</h1>
    <p>This is a simple web application.</p>
</body>
</html>
```

Setelah itu, kita akan mengisi `main.yml` di dalam `nginx/tasks/` dengan code berikut:
```yml
---
- name: Update apt cache
  apt:
    update_cache: yes
  become: true

- name: Install Nginx
  apt:
    name: nginx
    state: present
  become: true

- name: Copy webapp files
  copy:
    src: index.html
    dest: /var/www/html/
  become: true

- name: Ensure Nginx is started and enabled
  systemd:
    name: nginx
    state: started
    enabled: yes
  become: true

```
Berikut adalah penjelasan codenya:
1. Update apt cache:
- Task ini bertujuan untuk memperbarui cache paket menggunakan modul `apt` Ansible. Cache paket adalah penyimpanan sementara dari daftar paket yang tersedia yang disimpan di setiap mesin Debian atau Ubuntu.
- Dengan menetapkan `update_cache: yes`, Ansible akan menjalankan perintah `apt-get update` untuk memperbarui cache paket sebelum melanjutkan instalasi.
- Pengaturan `become: true` digunakan untuk mendapatkan akses superuser agar perintah dapat dijalankan dengan benar.

2. Install Nginx:
- Task ini menggunakan modul `apt` untuk menginstal paket Nginx pada sistem.
- Dengan menentukan `name: nginx`, Ansible tahu bahwa yang harus diinstal adalah paket dengan nama nginx.
- Pengaturan `state: present` menunjukkan bahwa kita ingin memastikan paket tersebut terinstal pada sistem.
- Sekali lagi, `become: true` digunakan untuk mendapatkan hak akses superuser yang diperlukan untuk menjalankan perintah instalasi.

3. Copy webapp files:
- Menggunakan modul copy untuk menyalin file `index.html` dari direktori lokal ke direktori `/var/www/html/` di server.
- `src: index.html` menunjukkan file yang akan disalin, sedangkan dest: `/var/www/html/` menunjukkan lokasi tujuan untuk penyalinan.
- Pengaturan `become: true` digunakan untuk hak akses superuser karena menulis ke direktori sistem.

4. Ensure Nginx is started and enabled:
- Task ini menggunakan modul `systemd` untuk memastikan bahwa layanan Nginx telah dimulai dan diaktifkan saat boot.
- Pengaturan `name: nginx` menentukan nama layanan yang akan diperiksa.
- `state: started` mengindikasikan bahwa kita ingin memastikan layanan tersebut telah dimulai.
- `enabled: yes` menunjukkan bahwa kita ingin layanan diaktifkan saat boot.
- Kembali, `become: true` digunakan untuk hak akses superuser yang diperlukan.


## Langkah 6: Jalankan Ansible
Dengan case sederhana yang telah dibuat, maka kita sudah cukup untuk menjalankan Ansible dengan command berikut:
```bash
ansible-playbook -i inventory.ini playbook.yml
```

<br>
<br>

Selain case di atas, kita juga bisa melakukan deployment Ansible dengan menggunakan backend pada modul di link berikut https://github.com/arsitektur-jaringan-komputer/modul-deployment/tree/master/2.%20Backend. Code dari deployment dengan menggunakan Ansible pada case tersebut dapat dilihat pada https://github.com/nabilaaidah/ansible-modul-2-be dan videonya dapat dilihat pada https://youtu.be/FNM4B8YfLOk?si=LGBdDSwVpgDWLK1M
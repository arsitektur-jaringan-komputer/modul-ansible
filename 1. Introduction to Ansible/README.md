<div align=center>

# Introduction to Ansible

</div>

## Table of Contents

## Table of Contents

1. [Introduction](#introduction)
2. [Other Tools Comparison](#other-tools-comparison)
3. [Installation](#installation)
4. [Inventory](#inventory)
   - [File Format](#file-format)
   - [Host Determination](#host-determination)
   - [Host Groups](#host-groups)
   - [Variables](#variables)
   - [Dynamic Extensions](#dynamic-extensions)
   - [Pattern Matching](#pattern-matching)
   - [Examples](#examples)
5. [Configuration File](#configuration-file)
   - [File Name and Location](#file-name-and-location)
   - [File Structure](#file-structure)
   - [Global Settings](#global-settings)
   - [Module Settings](#module-settings)
   - [Connection Settings](#connection-settings)
   - [Execution Settings](#execution-settings)
   - [Include Configuration](#include-configuration)
   - [Security](#security)
   - [Documentation](#documentation)
   - [Example](#example)
6. [Common Ad Hoc](#common-ad-hoc)
   - [Ping Hosts](#ping-hosts)
   - [Run Shell Command](#run-shell-command)
   - [Copy File](#copy-file)
   - [Install Package](#install-package)
   - [Restart Service](#restart-service)
   - [Remove File](#remove-file)
   - [View Host Information](#view-host-information)
   - [Run Script or Shell Command from File](#run-script-or-shell-command-from-file)
   - [Update Packages](#update-packages)
   - [Reboot Host](#reboot-host)

## Introduction

Ansible adalah platform manajemen konfigurasi dan otomatisasi open-source yang digunakan untuk menyederhanakan dan mempercepat pengelolaan infrastruktur IT. Berikut adalah beberapa poin utama tentang Ansible:

Deklaratif: Ansible menggunakan pendekatan deklaratif, di mana Anda mendefinisikan keadaan yang diinginkan untuk sistem Anda, bukan langkah-langkah spesifik untuk mencapainya.

Agentless: Ansible tidak memerlukan agent yang diinstal di host target. Ini berarti penerapan Ansible lebih ringan dan mudah dikonfigurasi.

YAML-based Playbooks: Konfigurasi dan tugas didefinisikan dalam file YAML yang mudah dibaca dan dimengerti oleh manusia.

Idempotent: Ansible Playbook secara alami idempoten, yang berarti Anda dapat menjalankannya berulang kali tanpa efek samping yang tidak diinginkan.

Komunitas yang Besar: Ansible memiliki komunitas yang besar dan aktif, yang berarti Anda dapat menemukan banyak dukungan, modul, dan contoh untuk digunakan.

## Other Tools Comparison

| Fitur          | Ansible                              | Chef                                 | Puppet                               | SaltStack                            |
|----------------|--------------------------------------|--------------------------------------|--------------------------------------|--------------------------------------|
| Model          | Deklaratif                           | Imperatif                            | Deklaratif                           | Campuran Deklaratif dan Imperatif    |
| Agent          | Agentless                            | Memerlukan agent                     | Memerlukan agent                     | Opsional: Agentless atau Memerlukan agent |
| Bahasa         | YAML                                 | Ruby                                 | Puppet DSL (Custom)                  | YAML, Python, Jinja2, dan PyDSL (Custom) |
| Skalabilitas   | Cocok untuk skala kecil hingga besar | Cocok untuk skala kecil hingga besar | Cocok untuk skala kecil hingga besar | Cocok untuk skala kecil hingga besar |
| Kurva Belajar  | Rendah                               | Tinggi                               | Tinggi                               | Sedang                               |
| Performa       | Cukup baik                           | Baik                                 | Baik                                 | Cukup baik                           |
| Komunitas      | Besar dan aktif                      | Besar dan aktif                      | Besar dan aktif                      | Besar dan aktif                      |
| Fleksibilitas  | Fleksibel dalam banyak lingkungan    | Kurang fleksibel dalam beberapa kasus| Fleksibel dalam banyak lingkungan    | Sangat fleksibel dalam banyak kasus  |

Perbandingan ini memberikan gambaran umum tentang perbedaan antara Ansible, Chef, Puppet, dan SaltStack dalam hal fitur utama, model kerja, dan karakteristik masing-masing. Keputusan untuk memilih alat yang sesuai tergantung pada kebutuhan dan preferensi spesifik dari proyek dan lingkungan Anda.

## Instalation

Lakukan Update dan Upgrade pada kernel anda
```bash
sudo apt-get update
sudo apt-get upgrade
```

Kemudian, install python dan pip
```bash
sudo apt-get install python3 python3-pip
```

Lalu, gunakan pip untuk menginstall ansible
```bash
pip3 install ansible
```
atau ansible-core
```bash
pip3 install ansible-core
```

Ansible dan Ansible Core adalah dua konsep yang berbeda:

1. **Ansible**: Ansible adalah platform manajemen konfigurasi dan otomatisasi open-source yang kuat yang mencakup berbagai fitur dan komponen untuk mengotomatisasi tugas-tugas IT, seperti provisioning, konfigurasi, penyebaran aplikasi, dan manajemen infrastruktur.

2. **Ansible Core**: Ansible Core adalah inti dari Ansible yang mencakup modul utama, perpustakaan, dan fungsi-fungsi dasar yang diperlukan untuk menjalankan Ansible. Ini adalah bagian utama dari Ansible dan menyediakan kerangka dasar untuk menjalankan tugas-tugas otomatisasi. Ansible Core tidak termasuk beberapa fitur tambahan yang disediakan dalam versi Ansible yang lebih lengkap.

Jadi, perbedaan utama antara Ansible dan Ansible Core adalah bahwa Ansible adalah platform lengkap yang mencakup berbagai fitur dan komponen tambahan, sementara Ansible Core adalah inti dari Ansible yang menyediakan kerangka dasar untuk menjalankan tugas-tugas otomatisasi.

## Inventory

Ansible inventory adalah sebuah file atau struktur data yang digunakan oleh Ansible untuk mengetahui target mana yang harus diatur, dikonfigurasi, atau dimanipulasi oleh perintah Ansible. Ini adalah bagian integral dari bagaimana Ansible mengelola infrastruktur dan perangkat lunak dalam lingkungan yang diotomatisasi. Berikut adalah beberapa poin detail tentang Ansible inventory:

### Format File
Inventory Ansible biasanya disimpan dalam file teks sederhana dengan format tertentu. Format ini dapat berupa file tunggal atau direktori dengan beberapa file, tergantung pada kompleksitas infrastruktur yang dikelola. File-file ini biasanya memiliki ekstensi .ini atau .yaml, tergantung pada preferensi dan kebutuhan pengguna.

<br>
Contoh inventory dengan format .ini

```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[loadbalancers]
lb1.example.com

```
<br>
Contoh inventory dengan format .yaml

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.0.101
        web2:
          ansible_host: 192.168.0.102
    databases:
      hosts:
        db1:
          ansible_host: 192.168.0.201
        db2:
          ansible_host: 192.168.0.202
```
<br>

Contoh Inventory bentuk Folder:
```bash
inventory/
    ├── group_vars/
    │   └── group1.yml
    ├── host_vars/
    │   └── host1.yml
    ├── production/
    │   ├── hosts
    │   ├── group_vars/
    │   │   └── prod_group.yml
    │   └── host_vars/
    │       └── prod_host.yml
    └── staging/
        ├── hosts
        ├── group_vars/
        │   └── stage_group.yml
        └── host_vars/
            └── stage_host.yml
```

Ansible inventory adalah komponen penting dalam ekosistem Ansible yang memungkinkan pengguna untuk secara efisien mengelola dan mengotomatisasi infrastruktur mereka. Dengan inventory yang baik, pengguna dapat dengan mudah menyesuaikan konfigurasi dan tindakan Ansible mereka sesuai dengan kebutuhan mereka.

### Penentuan Hosts
Inventory menentukan host atau node mana yang akan dikelola oleh Ansible. Ini bisa berupa alamat IP, nama host, atau alias yang diberikan.

### Grup Hosts
Hosts dalam inventory dapat dikelompokkan bersama berdasarkan kriteria tertentu. Misalnya, hosts dengan peran yang sama atau hosts yang berada di lokasi fisik yang sama dapat dikelompokkan bersama. Hal ini memungkinkan untuk melakukan tindakan yang sama pada sekelompok host secara bersamaan.

### Variabel
Inventory juga dapat menyimpan variabel yang terkait dengan setiap host atau kelompok host. Ini memungkinkan konfigurasi yang lebih dinamis dan fleksibel, karena variabel dapat digunakan dalam playbook Ansible untuk membuat konfigurasi yang bergantung pada kondisi atau lingkungan tertentu.

### Ekstensi Dinamis
Selain menggunakan file statis, Ansible juga mendukung inventory yang dibuat secara dinamis. Ini berarti Ansible dapat membangun inventorynya secara langsung dari sumber data yang berbeda, seperti cloud provider atau layanan konfigurasi lainnya.

### Penggunaan Pattern Matching
Ansible menyediakan pola pencocokan yang kuat untuk memilih host atau grup host dari inventory. Ini memungkinkan pengguna untuk menentukan target operasi Ansible dengan sangat fleksibel dan efisien.

### Example
```ini
# Global variables
ansible_user=admin
ansible_ssh_private_key=~/.ssh/id_rsa

# Group variables
[group1:vars]
group_variable1=value1
group_variable2=value2

# Host variables
host1 ansible_host=192.168.1.10 ansible_connection=ssh ansible_user=user1
host2 ansible_host=192.168.1.11 ansible_connection=ssh ansible_user=user2

# Conditional variables
[web_servers]
host1 http_port=80
host2 http_port=8080

[web_servers:vars]
http_port=8088

[web_servers:children]
web_servers_production

[web_servers_production]
host3 http_port=8081

[web_servers_production:vars]
http_port=8000

# Environment-specific variables
[production:vars]
environment=prod

[staging:vars]
environment=staging
```

## Config File

File konfigurasi Ansible, yang biasanya disebut sebagai ansible.cfg, adalah file yang digunakan untuk mengatur berbagai pengaturan dan opsi yang mengontrol perilaku Ansible secara keseluruhan. Ini memungkinkan pengguna untuk menyesuaikan berbagai aspek Ansible sesuai dengan kebutuhan mereka. Berikut adalah penjelasan detail tentang file konfigurasi Ansible:

### Nama File dan Lokasi
Secara default, file konfigurasi Ansible disebut ansible.cfg. Ansible mencari file ini dalam beberapa lokasi default, termasuk direktori saat ini, direktori /etc/ansible/, atau di direktori yang ditentukan oleh variabel lingkungan ANSIBLE_CONFIG. Lokasi dan nama file konfigurasi dapat disesuaikan sesuai dengan preferensi pengguna.

### Struktur File
File konfigurasi Ansible adalah file teks sederhana yang menggunakan format key = value. Ini berarti setiap pengaturan memiliki kunci dan nilai yang sesuai. Komentar dalam file konfigurasi dimulai dengan tanda #.

### Pengaturan Global
File konfigurasi Ansible dapat mengatur pengaturan global yang berlaku untuk semua eksekusi Ansible. Ini mencakup pengaturan seperti host SSH default, lokasi inventory default, dan opsi output yang disukai.

### Pengaturan Modul 
Pengguna dapat menentukan pengaturan default untuk modul Ansible tertentu dalam file konfigurasi. Ini memungkinkan pengguna untuk menetapkan nilai default untuk opsi modul, menghemat waktu dan upaya ketika menggunakan modul tersebut.

### Pengaturan Connection: 
Ansible dapat menggunakan berbagai jenis koneksi untuk berinteraksi dengan host yang ditargetkan. Pengguna dapat menentukan jenis koneksi default (misalnya, SSH atau WinRM) dan opsi terkait lainnya dalam file konfigurasi.

### Pengaturan Eksekusi
File konfigurasi Ansible dapat mengontrol cara Ansible mengeksekusi playbook dan perintah. Ini termasuk opsi seperti jumlah thread yang digunakan, timeout operasi, dan pengaturan verbose.

### Include Konfigurasi
File konfigurasi Ansible mendukung inklusi, yang memungkinkan pengguna untuk membagi konfigurasi mereka menjadi beberapa file yang lebih kecil dan terkelola dengan lebih mudah. Ini berguna untuk mengatur konfigurasi berdasarkan kriteria tertentu, seperti lingkungan atau proyek.

### Keamanan
File konfigurasi Ansible dapat mengatur opsi keamanan, seperti mengenkripsi kata sandi atau mengatur file inventory yang aman.

### Dokumentasi
Ansible menyertakan dokumentasi yang cukup lengkap tentang setiap opsi konfigurasi dalam file ansible.cfg. Ini memudahkan pengguna untuk memahami arti dan penggunaan setiap opsi.

<br>    
Contoh sederhana file konfigurasi Ansible:

```ini
[defaults]
inventory = /path/to/inventory
remote_user = ansible
private_key_file = ~/.ssh/id_rsa
host_key_checking = False
forks = 10

[ssh_connection]
pipelining = True
```

File konfigurasi Ansible memberikan pengguna kekuatan untuk menyesuaikan perilaku Ansible sesuai dengan kebutuhan mereka, memungkinkan untuk penggunaan yang lebih efisien dan fleksibel dalam mengotomatisasi infrastruktur dan tugas-tugas pengelolaan sistem.

## Common Ad Hoc

Ad hoc commands dalam Ansible adalah perintah yang dieksekusi secara langsung dari baris perintah tanpa menggunakan playbook. Ini memungkinkan administrator sistem untuk melakukan tindakan cepat dan sederhana pada host atau kelompok host tanpa harus menulis playbook terlebih dahulu. Berikut adalah beberapa contoh ad hoc commands Ansible yang sering digunakan:

1. **Ping Hosts**:
   ```bash
   ansible all -m ping
   ```
   Perintah ini mengirimkan ping ke semua host yang terdaftar dalam inventory Ansible dan memberikan respons berupa pong dari host yang aktif.

2. **Menjalankan Perintah Shell**:
   ```bash
   ansible all -a "/bin/echo hello"
   ```
   Ini akan menjalankan perintah `/bin/echo hello` pada semua host yang terdaftar dan mengembalikan hasilnya.

3. **Copy File**:
   ```bash
   ansible all -m copy -a "src=/path/to/src/file dest=/path/to/destination/file"
   ```
   Perintah ini menyalin file dari host pengendali ke host yang ditentukan.

4. **Install Paket**:
   ```bash
   ansible all -m yum -a "name=<package_name> state=installed"
   ```
   Ini akan menginstal paket pada semua host yang menggunakan manajer paket Yum (untuk sistem operasi Linux berbasis Red Hat) atau:
   ```bash
   ansible all -m apt -a "name=<package_name> state=installed"
   ```
   Untuk menginstal paket pada host yang menggunakan manajer paket APT (untuk sistem operasi Linux berbasis Debian).

5. **Restart Service**:
   ```bash
   ansible webservers -m service -a "name=httpd state=restarted"
   ```
   Perintah ini akan merestart layanan HTTP pada host yang tergabung dalam grup `webservers`.

6. **Menghapus File**:
   ```bash
   ansible all -m file -a "path=/path/to/file state=absent"
   ```
   Ini akan menghapus file yang ditentukan dari semua host yang terdaftar.

7. **Melihat Informasi Host**:
   ```bash
   ansible all -m setup
   ```
   Perintah ini akan menampilkan informasi sistem dari semua host yang terdaftar dalam inventory, termasuk informasi tentang hardware, jaringan, dan lingkungan host.

8. **Menjalankan Script atau Perintah Shell dari File**:
   ```bash
   ansible all -a "/path/to/script.sh"
   ```
   Perintah ini menjalankan skrip bash atau perintah shell dari file pada semua host yang terdaftar.

9. **Melakukan Pembaruan Paket**:
   ```bash
   ansible all -m yum -a "name=* state=latest"
   ```
   Ini akan memperbarui semua paket pada host yang menggunakan manajer paket Yum.

10. **Melakukan Reboot Host**:
    ```bash
    ansible all -a "/sbin/reboot"
    ```
    Perintah ini akan merestart semua host yang terdaftar dalam inventory.

Ad hoc commands adalah alat yang sangat berguna untuk melakukan tindakan cepat dan sederhana pada infrastruktur yang dikelola dengan Ansible tanpa perlu menulis playbook terlebih dahulu. Ini memungkinkan administrator sistem untuk dengan mudah melakukan tugas-tugas pengelolaan harian dengan cepat dan efisien.
# Advance Ansible Playbook
Setelah mempelajari ansible playbook pada modul sebelumnya, kita akan mempelajari lebih mengenai playbook pada modul ini juga. Dalam ansible, terdapat dua materi, seperti role dan vault.

## Table of Contents

1. [Ansible Role](#ansible-role)
2. [Ansible Vault](#ansible-vault)

## Ansible Role
Ansible role merupakan salah satu konsep utama dalam Ansible yang memungkinkan pengguna untuk mengelola dan mengorganisir konfigurasi dan tugas secara modular. Sebuah role adalah seperangkat tugas, variabel, file konfigurasi, dan modul Ansible yang terstruktur secara hierarki untuk melakukan fungsi-fungsi tertentu pada host atau grup host yang ditentukan. Penggunaan role membantu dalam pengelolaan konfigurasi sistem dengan memecahnya menjadi bagian-bagian yang lebih kecil dan terkelola.


Kita dapat melakukan generate template role dengan menggunakan command berikut
```
ansible-galaxy init <my_role>
```
Pada command di atas digunakan ansible galaxy yang merupakan pository berbagi berbagai role Ansible yang dapat digunakan oleh komunitas untuk mengelola konfigurasi.

Hasil template dari command berikut adalah:
```
my_role/
├── defaults/
├── files/
├── handlers/
├── meta/
├── tasks/
├── templates/
└── vars/
```

Keterangan dari folder-folder tersebut adalah seperti berikut:

#### 1. defaults
 Berisi variabel default untuk role.
 
 Berikut adalah contoh code `main.yml`, yang umumnya digunakan di dalam folder `defaults`:
 ```
 ---
nginx_port: 80
nginx_root: /var/www/html
 ```
 Pada code di atas, terisi variabel-variabel yang dapat diakses dalam playbook atau task lain di dalam role.

<br>

Berikut adalah cara mengakses variabel yang ada pada `main.yml` di dalam:

1. Playbook

Pada playbook, kita harus mendefinisikan path dari defaults/main.yml, sehingga codenya akan terlihat seperti ini:
```
# playbook.yml

- hosts: web_servers
  vars_files:
    - roles/my_role/defaults/main.yml
  tasks:
    - name: Display nginx_port variable
      debug:
        msg: "The nginx_port variable is {{ nginx_port }}"

    - name: Display nginx_root variable
      debug:
        msg: "The nginx_root variable is {{ nginx_root }}"

    - name: Another task
      debug:
        msg: "Another task that uses the variables from defaults/main.yml"
```

2. Task Lain Pada Role Sama

Pada task lain dengan role yang sama, tidak perlu mendefinisikan path dari defaults/main.yml, sehingga codenya akan terlihat seperti berikut:
```
# roles/my_role/tasks/another_task.yml

- name: Another task in the same role
  debug:
    msg: "The nginx_port variable is {{ nginx_port }}"
```


#### 2. files
Tempat untuk menyimpan file yang akan di-copy ke target host.

Berikut adalah contoh struktur di dalam dir `files`:
```
# Contoh file konfigurasi nginx
files/
└── nginx.conf
```
Pada contoh di atas, files dapat digunakan sebagai tempat menyimpan konfigurasi nginx.

<br>

Berikut adalah cara mengakses file yang ada di dalam files:
```
- name: Copy and template file from files/ directory
  template:
    src: files/my_template.j2
    dest: /path/to/destination/my_template.txt
```
Dalam contoh ini, file `my_template.j2` dianggap sebagai template Jinja2 yang berada dalam direktori `files/`. Ansible akan merender template tersebut dan menyalin hasilnya ke `/path/to/destination/my_template.txt` pada target host.


#### 3. handlers
Handler adalah tindakan yang akan dilakukan jika task sebelumnya menghasilkan perubahan.

Berikut adalah contoh `main.yml` di dalam folder `handlers`:
```
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```
Handler akan dijalankan jika ada task yang memodifikasi konfigurasi nginx, seperti mengubah file konfigurasi.

<br>

Berikut adalah cara memanggil `handlers` pada `playbook` dengan menggunakan notify:
```
# playbook.yml

- hosts: web_servers
  tasks:
    - name: Copy Nginx configuration file
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx
```
Dalam contoh di atas, ketika tugas "Copy Nginx configuration file" dijalankan, itu juga akan memicu handler `Restart Nginx` yang telah didefinisikan sebelumnya dalam file `handlers/main.yml`. Handler tersebut akan menyalakan kembali layanan Nginx setelah konfigurasinya diperbarui.


#### 4. meta
File meta/main.yml berisi metadata role seperti dependencies.

Berikut adalah contoh `main.yml` di dalam folder `meta`:
```
---
dependencies:
  - role: common
```
Ini menunjukkan bahwa role ini memiliki dependensi terhadap role "common".

<br>

Contoh terdapat case seperti berikut:
```
roles/
├── database
│   ├── tasks
│   │   └── main.yml
│   └── meta
│       └── main.yml
└── webserver
    ├── tasks
    │   └── main.yml
    └── meta
        └── main.yml
```

Dengan di dalam adalah `roles/webserver/meta/main.yml` sebagai berikut:
```
---
dependencies:
  - role: database
```

Maka, role `webserver` memiliki dependensi terhadap role `database`, yang berarti role `database` akan dijalankan sebelum role `webserver`.

#### 5. tasks
Berisi task-tasks yang akan dijalankan oleh role. 

Berikut adalah contoh `main.yml` di dalam folder `tasks`:
```
---
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Ensure nginx service is running
  service:
    name: nginx
    state: started
    enabled: yes
```
Ini adalah contoh task yang menginstal nginx dan memastikan bahwa layanan nginx berjalan.

#### 6. templates
Berisi template files yang akan di-render dan di-copy ke target host.

Berikut adalah contoh struktur di dalam dir `templates`:
```
# Contoh file template untuk konfigurasi nginx
templates/
└── nginx.conf.j2
```
Isi dari file template ini akan di-render sesuai dengan nilai variabel yang diberikan.

#### 7. tests
Berisi file-file untuk melakukan testing terhadap role. 

Berikut adalah contoh `test.yml` di dalam folder `tests`:
```
# roles/my_role/tests/test.yml

- hosts: localhost
  tasks:
    - name: Check if Nginx configuration file is present
      stat:
        path: /etc/nginx/nginx.conf
      register: nginx_conf_file

    - name: Print Nginx configuration file status
      debug:
        msg: "Nginx configuration file is {{ 'present' if nginx_conf_file.stat.exists else 'absent' }}"

    - name: Check if Nginx service is running
      service_facts:
      register: services

    - name: Print Nginx service status
      debug:
        msg: "Nginx service is {{ 'running' if services.services['nginx'].state == 'running' else 'stopped' }}"
```

Dan untuk testingnya dapat dijalankan dengan command berikut:
```
ansible-playbook -i localhost, roles/my_role/tests/test.yml
```

#### 8. vars
Berisi variabel-variabel tambahan yang digunakan oleh role.

Berikut adalah contoh `main.yml` di dalam folder `vars`:
```
---
nginx_user: www-data
```
Variabel ini dapat digunakan di task-task atau playbook yang menggunakan role ini.

### Perbedaan antara files dan templates
1. Direktori templates:
- Digunakan untuk menyimpan file-file yang berpotensi menjadi template Jinja2.
- File-file dalam direktori templates biasanya berupa file konfigurasi yang memerlukan pengubahan nilai-nilai variabel berdasarkan konteks atau kondisi tertentu.
- File-file dalam direktori templates akan di-render oleh Ansible menggunakan mesin template Jinja2 sebelum disalin atau diaplikasikan ke host target.
- Template Jinja2 memungkinkan untuk menambahkan logika atau pengubahan yang dinamis ke dalam file konfigurasi, seperti penggunaan variabel, perulangan, percabangan, dan lain-lain.

2. Direktori files:
- Digunakan untuk menyimpan file-file statis yang akan disalin langsung ke host target tanpa perubahan atau pengubahan apapun.
- File-file dalam direktori files tidak akan di-render atau dimodifikasi oleh Ansible sebelum disalin.
- Biasanya, file-file dalam direktori files adalah file-binari, script, atau file-file konfigurasi yang tidak memerlukan pengubahan nilai-nilai variabel.


<br>

Dalam pengerjaannya, ansible akan mengeksekusi role dari urutan paling atas. Pada contoh gambar di bawah ini, ansible akan menjalankan update, lalu nginx, lalu mysql, dan terakhir adalah backend

<img width="96" alt="Screenshot 2024-03-02 at 10 30 30" src="https://github.com/arsitektur-jaringan-komputer/modul-ansible/assets/110476969/ecc1ce0b-dd7a-463c-ae98-3c365a9ae773">

</br>


## Ansible Vault

Ansible Vault adalah fitur dalam Ansible yang digunakan untuk mengenkripsi data sensitif seperti kata sandi, kunci SSH, atau informasi rahasia lainnya dalam file Ansible playbooks atau variabel. Tujuan utamanya adalah untuk melindungi informasi rahasia ini agar tidak terbaca secara langsung oleh orang yang tidak berwenang.

Berikut adalah poin penting mengenai ansible vault:

1. Enkripsi Data Rahasia:

Menggunakan Ansible Vault untuk mengamankan data sensitif dalam file Ansible.
Menggunakan algoritma enkripsi yang kuat, seperti AES, untuk melindungi informasi dari akses yang tidak sah.

2. Integrasi dengan Ansible Playbooks:

Mengenkripsi variabel, file variabel, atau seluruh playbook Ansible.
Memungkinkan penyimpanan kata sandi, kunci SSH, atau informasi rahasia lainnya dalam playbook tanpa risiko akses yang tidak sah.

3. Kompatibilitas dengan Berbagai Algoritma Enkripsi:

Ansible Vault mendukung berbagai algoritma enkripsi, termasuk AES dengan kunci 256-bit.
Fleksibilitas dalam memilih tingkat keamanan sesuai dengan kebutuhan.

4. Pengelolaan Kata Sandi:

Memungkinkan pengelolaan kata sandi untuk proses enkripsi dan dekripsi.
Mendukung penggunaan kata sandi tunggal atau kunci enkripsi SSH sebagai metode otentikasi.

5. Perintah dan Utilitas:

Ansible menyediakan perintah dan utilitas khusus untuk membuat, mengedit, mengenkripsi, dan mendekripsi file yang dienkripsi oleh Ansible Vault.
Memudahkan pengelolaan dan interaksi dengan data yang dienkripsi.

6. Deklaratif dan Terdokumentasi:

Penggunaan Ansible Vault dideklarasikan dalam playbook Ansible.
Memastikan penggunaan yang mudah dipahami dan didokumentasikan.

7. Integrasi dengan Sumber Kontrol:

File yang dienkripsi oleh Ansible Vault dapat dengan aman dimasukkan ke dalam repositori sumber kontrol seperti Git.
Mendukung kolaborasi tim dalam proyek Ansible tanpa mengorbankan keamanan informasi rahasia.


Untuk menjalankan ansible vault, dapat digunakan command berikut:
```
ansible-vault create vault.yml
```

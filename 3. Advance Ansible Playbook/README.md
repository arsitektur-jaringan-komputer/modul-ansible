# Advance Ansible Playbook
Setelah mempelajari ansible playbook pada modul sebelumnya, kita akan mempelajari lebih mengenai playbook pada modul ini juga. Dalam ansible, terdapat dua materi, seperti role dan vault.

## Table of Contents

1. [Ansible Role](#ansible-role)
2. [Ansible Vault](#ansible-vault)

## Ansible Role
Ansible role merupakan salah satu konsep utama dalam Ansible yang memungkinkan pengguna untuk mengelola dan mengorganisir konfigurasi dan tugas secara modular. Sebuah role adalah seperangkat tugas, variabel, file konfigurasi, dan modul Ansible yang terstruktur secara hierarki untuk melakukan fungsi-fungsi tertentu pada host atau grup host yang ditentukan. Penggunaan role membantu dalam pengelolaan konfigurasi sistem dengan memecahnya menjadi bagian-bagian yang lebih kecil dan terkelola.


Kita dapat melakukan generate template role dengan menggunakan command berikut
```bash
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
 ```yaml
 ---
nginx_port: 80
nginx_root: /var/www/html
 ```
 Pada code di atas, terisi variabel-variabel yang dapat diakses dalam playbook atau task lain di dalam role.

<br>

Berikut adalah cara mengakses variabel yang ada pada `defaults/main.yml` di dalam:

1. Playbook

Pada playbook, kita harus mendefinisikan path dari defaults/main.yml, sehingga codenya akan terlihat seperti ini:
```yaml
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
```yaml
# roles/my_role/tasks/another_task.yml

- name: Another task in the same role
  debug:
    msg: "The nginx_port variable is {{ nginx_port }}"
```


#### 2. files
Tempat untuk menyimpan file yang akan di-copy ke target host.

Berikut adalah contoh struktur di dalam dir `files`:
```bash
# Contoh file konfigurasi nginx
files/
└── nginx.conf
```
Pada contoh di atas, files dapat digunakan sebagai tempat menyimpan konfigurasi nginx.

<br>

Berikut adalah cara mengakses file yang ada di dalam files:
```yaml
- name: Copy and template file from files/ directory
  template:
    src: files/nginx.conf
    dest: /path/to/destination/my_template.txt
```
Dalam contoh ini, file `nginx.conf` yang berada dalam direktori `files/` akan dirender dan disalin hasilnya ke `/path/to/destination/my_template.txt` pada target host.


#### 3. handlers
Handler adalah tindakan yang akan dilakukan jika task sebelumnya menghasilkan perubahan.

Berikut adalah contoh `main.yml` di dalam folder `handlers`:
```yaml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```
Handler akan dijalankan jika ada task yang memodifikasi konfigurasi nginx, seperti mengubah file konfigurasi.

<br>

Berikut adalah cara memanggil `handlers` pada `playbook` dengan menggunakan `notify`:
```yaml
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
```yaml
---
dependencies:
  - role: common
```
Ini menunjukkan bahwa role ini memiliki dependensi terhadap role "common".

<br>

Contoh terdapat case seperti berikut:
```bash
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

Dengan di dalam `roles/webserver/meta/main.yml` sebagai berikut:
```yaml
---
dependencies:
  - role: database
```

Maka, role `webserver` memiliki dependensi terhadap role `database`, yang berarti role `database` akan dijalankan sebelum role `webserver`.

#### 5. tasks
Berisi task-tasks yang akan dijalankan oleh role. 

Berikut adalah contoh `main.yml` di dalam folder `tasks`:
```yaml
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
```bash
# Contoh file template untuk konfigurasi nginx
templates/
└── nginx.conf.j2
```
Isi dari file template ini akan di-render sesuai dengan nilai variabel yang diberikan.

#### 7. tests
Berisi file-file untuk melakukan testing terhadap role. 

Berikut adalah contoh `test.yml` di dalam folder `tests`:
```yaml
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
```bash
ansible-playbook -i localhost, roles/my_role/tests/test.yml
```

#### 8. vars
Berisi variabel-variabel tambahan yang digunakan oleh role.

Berikut adalah contoh `main.yml` di dalam folder `vars`:
```yaml
---
nginx_user: www-data
```
Variabel ini dapat digunakan di task-task atau playbook yang menggunakan role ini.

### Perbedaan antara files dan templates
<b> 1. Direktori templates: </b>
- Digunakan untuk menyimpan file-file yang berpotensi menjadi template Jinja2.
- File-file dalam direktori templates biasanya berupa file konfigurasi yang memerlukan pengubahan nilai-nilai variabel berdasarkan konteks atau kondisi tertentu.
- File-file dalam direktori templates akan di-render oleh Ansible menggunakan mesin template Jinja2 sebelum disalin atau diaplikasikan ke host target.
- Template Jinja2 memungkinkan untuk menambahkan logika atau pengubahan yang dinamis ke dalam file konfigurasi, seperti penggunaan variabel, perulangan, percabangan, dan lain-lain.

<b> 2. Direktori files: </b>
- Digunakan untuk menyimpan file-file statis yang akan disalin langsung ke host target tanpa perubahan atau pengubahan apapun.
- File-file dalam direktori files tidak akan di-render atau dimodifikasi oleh Ansible sebelum disalin.
- Biasanya, file-file dalam direktori files adalah file-binari, script, atau file-file konfigurasi yang tidak memerlukan pengubahan nilai-nilai variabel.

### Perbedaan antara defaults dan vars
<b> 1. Direktori defaults: </b>
- Digunakan untuk menyimpan variabel default yang akan digunakan oleh role jika tidak ada nilai variabel yang diberikan oleh pengguna.
- Variabel yang didefinisikan dalam direktori defaults dapat diubah oleh pengguna saat playbook dijalankan, baik melalui file playbook, group_vars, host_vars, atau secara langsung melalui parameter command-line.
- Variabel yang didefinisikan di dalam defaults/main.yml akan dianggap sebagai nilai default yang akan digunakan jika variabel tersebut tidak ditetapkan dengan nilai yang sesuai oleh pengguna saat menjalankan playbook.

<b> 2. Direktori vars: </b>
- Digunakan untuk menyimpan variabel yang spesifik untuk role dan tidak dianggap sebagai nilai default.
- Variabel yang didefinisikan dalam direktori vars tidak dapat diubah oleh pengguna saat playbook dijalankan.
- Variabel yang didefinisikan di dalam vars/main.yml akan dianggap sebagai nilai yang tetap dan tidak akan berubah selama playbook dijalankan, kecuali jika nilai tersebut ditimpa oleh variabel yang didefinisikan di dalam playbook atau file lain yang memiliki tingkat prioritas yang lebih tinggi.


<br>

Dalam pengerjaannya, ansible akan mengeksekusi role dari urutan paling atas. Pada contoh gambar di bawah ini, ansible akan menjalankan update, lalu nginx, lalu mysql, dan terakhir adalah backend

<img width="96" alt="Screenshot 2024-03-02 at 10 30 30" src="https://github.com/arsitektur-jaringan-komputer/modul-ansible/assets/110476969/ecc1ce0b-dd7a-463c-ae98-3c365a9ae773">

</br>


## Ansible Vault

Ansible Vault adalah fitur dalam Ansible yang digunakan untuk mengenkripsi data sensitif seperti kata sandi, kunci SSH, atau informasi rahasia lainnya dalam file Ansible playbooks atau variabel. Tujuan utamanya adalah untuk melindungi informasi rahasia ini agar tidak terbaca secara langsung oleh orang yang tidak berwenang.

<br>

### Berikut adalah contoh penggunaan Ansible Vault:
1. Membuat File Vault
Untuk membuat file vault baru
```bash
ansible-vault create secret.yml
```
Pada saat menjalankan command ini, kalian akan diminta untuk memasukkan kata sandi dan kemudian membuka file teks untuk diedit dengan menggunakan editor default.

2. Mengedit File Vault
Untuk mengedit file vault yang ada
```bash
ansible-vault edit secret.yml
```
Command ini akan meminta kata sandi yang telah kalian buat saat melakukan create Ansible Vault dan membuka file dalam editor untuk diedit.

3. Mengenkripsi File Vault
Untuk mengenkripsi file yang sudah ada
```bash
ansible-vault encrypt secret.yml
```

4. Dekripsi File Vault
Untuk mendekripsi file vault
```bash
ansible-vault decrypt secret.yml
```

5. Menjalankan Playbook dengan Vault
Saat menjalankan playbook, dapat menyertakan opsi --ask-vault-pass untuk meminta kata sandi vault
```bash
ansible-playbook playbook.yml --ask-vault-pass
```

<br>

### Contoh case penggunaan Ansible Vault:
1. Persiapan Vault
Buat file Ansible Vault untuk menyimpan kata sandi database.
```bash
ansible-vault create secrets.yml
``` 
Lalu, masukkan password yang akan digunakan dalam membuka Ansible Vault

2. Isi Vault
Dalam file `secrets.yml`, tambahkan entri untuk kata sandi database
```yaml
database_password: your_database_password
```
Simpan perubahan dan tutup file.

3. Gunakan dalam Playbook
Dalam playbook Ansible, dapat digunakan variabel dari Ansible Vault
```yaml
- name: Konfigurasi Database
  hosts: database_servers
  tasks:
    - name: Set database password
      mysql_user:
        name: myuser
        password: "{{ vaulted_database_password }}"
  vars_files:
    - secrets.yml
```
Dalam contoh ini, `vaulted_database_password` adalah variabel yang mengambil nilai dari Ansible Vault.

4. Menjalankan Playbook
Ketika menjalankan playbook, Ansible akan meminta kata sandi vault
```bash
ansible-playbook playbook.yml --ask-vault-pass
```
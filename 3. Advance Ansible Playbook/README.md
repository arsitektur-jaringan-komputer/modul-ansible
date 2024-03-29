# Advance Ansible Playbook
Setelah mempelajari ansible playbook pada modul sebelumnya, kita akan mempelajari lebih mengenai playbook pada modul ini juga. Dalam ansible, terdapat dua materi, seperti role dan vault.

## Table of Contents

1. [Playbook](#playbook)
2. [Variable](#variable)
3. [Condition](#condition)
4. [Looping](#looping)
4. [Handlers](#handlers)

### Ansible Role
Ansible role merupakan salah satu konsep utama dalam Ansible yang memungkinkan pengguna untuk mengelola dan mengorganisir konfigurasi dan tugas secara modular. Sebuah role adalah seperangkat tugas, variabel, file konfigurasi, dan modul Ansible yang terstruktur secara hierarki untuk melakukan fungsi-fungsi tertentu pada host atau grup host yang ditentukan. Penggunaan role membantu dalam pengelolaan konfigurasi sistem dengan memecahnya menjadi bagian-bagian yang lebih kecil dan terkelola.


Kita dapat melakukan generate template role dengan menggunakan command berikut
```
ansible-galaxy init nama_role
```
Pada command di atas digunakan ansible galaxy yang merupakan pository berbagi berbagai role Ansible yang dapat digunakan oleh komunitas untuk mengelola konfigurasi.


Hasil template dari command berikut adalah:

<img width="212" alt="Screenshot 2024-03-02 at 10 28 53" src="https://github.com/arsitektur-jaringan-komputer/modul-ansible/assets/110476969/6c233e93-e4e2-45a3-9682-b64475eca2ed">


Keterangan dari folder-folder tersebut adalah seperti berikut:

Ketika Anda menggunakan perintah ansible-galaxy init untuk membuat template role dari Ansible Galaxy, Ansible akan membuat struktur direktori dan file yang umumnya diperlukan untuk role. Berikut adalah penjelasan singkat tentang struktur template yang digenerate:

1. defaults/: Direktori ini berisi file main.yml yang dapat digunakan untuk menyimpan variabel default untuk role. Variabel-variabel ini dapat diubah oleh pengguna ketika role digunakan dalam playbook.

2. files/: Direktori ini dapat digunakan untuk menyimpan file yang diperlukan oleh role dan perlu disalin ke host target. Misalnya, file konfigurasi atau skrip eksekusi.

3. handlers/: Direktori ini berisi file main.yml yang berisi definisi handler. Handler adalah tugas-tugas yang dijalankan sebagai respons terhadap perubahan konfigurasi dan biasanya digunakan untuk mengaktifkan ulang layanan atau tindakan serupa.

4. meta/: Direktori ini berisi file main.yml yang digunakan untuk menyimpan metadata role. Metadata ini dapat mencakup informasi seperti dependensi role atau penjelasan singkat tentang role tersebut.

5. tasks/: Direktori ini berisi file main.yml yang memuat tugas-tugas utama yang akan dijalankan oleh role. Tugas-tugas ini mungkin termasuk instalasi paket, konfigurasi file, atau tugas-tugas lain yang berkaitan dengan role.

6. templates/: Direktori ini dapat digunakan untuk menyimpan template file konfigurasi yang akan disesuaikan dengan variabel yang didefinisikan oleh pengguna.

7. tests/: Direktori ini dapat digunakan untuk menyimpan skrip atau playbook yang membantu dalam menguji role.

8. vars/: Direktori ini dapat digunakan untuk menyimpan file main.yml yang berisi variabel yang diperlukan oleh role. Variabel ini dapat digunakan untuk menyimpan konfigurasi yang bersifat statis dan dapat diubah oleh pengguna.



Dalam pengerjaannya, ansible akan mengeksekusi role dari urutan paling atas. Pada contoh gambar di bawah ini, ansible akan menjalankan update, lalu nginx, lalu mysql, dan terakhir adalah backend

<img width="96" alt="Screenshot 2024-03-02 at 10 30 30" src="https://github.com/arsitektur-jaringan-komputer/modul-ansible/assets/110476969/ecc1ce0b-dd7a-463c-ae98-3c365a9ae773">


## Ansible Vault

Ansible Vault adalah fitur dalam Ansible yang digunakan untuk mengenkripsi data sensitif seperti kata sandi, kunci SSH, atau informasi rahasia lainnya dalam file Ansible playbooks atau variabel. Tujuan utamanya adalah untuk melindungi informasi rahasia ini agar tidak terbaca secara langsung oleh orang yang tidak berwenang.

Berikut adalah poin penting mengenai ansible vault:

1. Enkripsi Data Rahasia:

Menggunakan Ansible Vault untuk mengamankan data sensitif dalam file Ansible.
Menggunakan algoritma enkripsi yang kuat, seperti AES, untuk melindungi informasi dari akses yang tidak sah.

2.Integrasi dengan Ansible Playbooks:

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

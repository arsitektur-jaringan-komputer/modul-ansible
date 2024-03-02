<div align=center>

# Ansible Playbook

</div>

## Table of Contents

## Table of Contents

1. [Playbook](#playbook)
2. [Variable](#variable)
   - [Variable Types](#variable-types)
   - [Defining Variables](#defining-variables)
   - [Variable Scope](#variable-scope)
   - [Referencing Variables](#referencing-variables)
   - [Variable Storage](#variable-storage)
   - [Default Variables](#default-variables)
   - [Example](#example-1)
3. [Condition](#condition)
    - [Types of Conditions](#types-of-conditions)
    - [Condition Syntax](#condition-syntax)
    - [Condition Expressions](#condition-expressions)
    - [Referencing Conditions](#referencing-conditions)
    - [Special Conditions](#special-conditions)
    - [Example](#example-2)
4. [Looping](#looping)
    - [Types of Looping](#types-of-looping)
    - [Extensive Usage](#extensive-usage)
    - [Example](#example-3)
5. [Handlers](#handlers)
    - [Usage of Handlers](#usage-of-handlers)
    - [Advantages of Using Handlers](#advantages-of-using-handlers)
    - [Extensive Usage](#extensive-usage-1)
    - [Example](#example-4)

## Playbook

Ansible playbook adalah file YAML yang berisi serangkaian instruksi untuk menjalankan tugas-tugas otomatis pada satu atau lebih host. Setiap tugas dapat mencakup modul Ansible yang menjalankan perintah, mengelola file, atau melakukan tindakan lainnya. Playbook memungkinkan otomatisasi konfigurasi, penyebaran aplikasi, dan manajemen infrastruktur dengan mudah dan konsisten melalui Ansible. Mereka juga mendukung variabel, pengulangan, dan struktur kontrol lainnya untuk menangani skenario yang lebih kompleks.

### Example
```yaml
---
- name: Contoh Playbook Ansible
  hosts: servers
  vars:
    http_port: 80
    app_servers:
      - server1
      - server2
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present
      become: yes

    - name: Copy index.html
      template:
        src: index.html.j2
        dest: /var/www/html/index.html
      notify: Restart Apache

    - name: Start Apache
      service:
        name: httpd
        state: started
      become: yes

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted
      become: yes
```

## Variable

Dalam playbook Ansible, variabel adalah nilai yang dapat diubah atau diatur agar digunakan oleh tugas-tugas yang didefinisikan dalam playbook. Variabel memungkinkan Anda untuk membuat playbook yang lebih dinamis dan dapat digunakan kembali dengan mengisolasi nilai-nilai yang mungkin berubah di satu tempat. Berikut adalah beberapa detail lebih lanjut tentang variabel dalam playbook Ansible:

### Tipe Variabel:

* **Variabel Lingkup Global**: Variabel yang didefinisikan di tingkat global dalam playbook atau di file yang diimpor.
* **Variabel Lingkup Per Tugas**: Variabel yang didefinisikan dalam konteks tugas tertentu dan hanya tersedia untuk tugas tersebut.
* **Variabel Lingkup Host**: Variabel yang didefinisikan untuk host tertentu dalam inventory Ansible.
* **Variabel Lingkup Grup**: Variabel yang didefinisikan untuk grup host dalam inventory Ansible.

### Cara Mendefinisikan Variabel:

1. **Inline**: Variabel dapat didefinisikan secara langsung dalam tugas dengan menggunakan sintaks var_name: value.
Menggunakan file var: Variabel dapat didefinisikan dalam file var yang dimuat dengan menggunakan direktif vars_files.
Menggunakan Role: Variabel dapat didefinisikan dalam role Ansible.
2. **Inventory**: Variabel dapat didefinisikan dalam file inventory Ansible.
3. **Prompt User**: Ansible juga memungkinkan Anda untuk meminta pengguna untuk memasukkan nilai variabel selama runtime playbook.

### Prioritas Variabel:

Ansible memiliki urutan prioritas untuk variabel. Nilai yang ditetapkan pada level yang lebih tinggi dalam hierarki akan menimpa nilai yang ditetapkan pada level yang lebih rendah.
Urutan prioritasnya adalah: variabel per tugas, variabel per host, variabel per grup, dan variabel global.

### Referensi Variabel:

Variabel dapat dirujuk dalam playbook menggunakan sintaks `{{ var_name }}`.
Variabel juga dapat dirujuk menggunakan ekspresi Jinja2 untuk memungkinkan operasi lebih kompleks.

### Penyimpanan Variabel:

Ansible menyimpan variabel dalam struktur data yang disebut 'Fakt', yang mencakup informasi tentang host, grup, dan fakta lainnya yang dihasilkan oleh Ansible saat menjalankan playbook.

### Variabel Bawaan:

Ansible menyertakan beberapa variabel bawaan yang berisi informasi tentang host, grup, sistem, dan lingkungan di mana playbook dijalankan.

Dengan memahami konsep dan penggunaan variabel ini, Anda dapat membuat playbook Ansible yang lebih fleksibel dan dapat digunakan kembali untuk mengelola infrastruktur secara efisien.

### Example

1.
```yaml
# Variables example
- name: Print variable types and kinds
  hosts: localhost
  gather_facts: no
  vars:
    # Simple variable
    my_var: "Hello, Ansible!"
    
    # List variable
    my_list:
      - apple
      - banana
      - orange
      
    # Dictionary variable
    my_dict:
      fruit: apple
      color: red
      
    # Variable with nested data structure
    my_nested_dict:
      fruits:
        - name: apple
          color: red
        - name: banana
          color: yellow
      
    # Variable with a boolean value
    my_bool: true
    
    # Variable with an integer value
    my_int: 42
    
    # Variable with a float value
    my_float: 3.14
    
  tasks:
    - name: Print variable types and values
      debug:
        msg: |
          my_var: {{ my_var }}
          my_list: {{ my_list }}
          my_dict: {{ my_dict }}
          my_nested_dict: {{ my_nested_dict }}
          my_bool: {{ my_bool }}
          my_int: {{ my_int }}
          my_float: {{ my_float }}
```
2. 
```yaml
# Variable scope example
- name: Variable Scope Example
  hosts: localhost
  gather_facts: no
  vars:
    global_var: "I'm a global variable"

  tasks:
    - name: Task 1 - Using global variable
      debug:
        msg: "Global variable: {{ global_var }}"

    - name: Task 2 - Defining a playbook variable
      vars:
        playbook_var: "I'm a playbook variable"
      debug:
        msg: "Playbook variable: {{ playbook_var }}"

    - name: Task 3 - Using playbook variable in another task
      debug:
        msg: "Playbook variable in another task: {{ playbook_var }}"

    - name: Task 4 - Defining a task variable
      debug:
        msg: "Task variable: {{ task_var }}"
      vars:
        task_var: "I'm a task variable"
```

3. 
```yaml
---
- name: Prompt User Variable Example
  hosts: localhost
  gather_facts: false
  vars_prompt:
    - name: user_name
      prompt: "Please enter your name:"
  tasks:
    - name: Display User Name
      debug:
        msg: "Hello, {{ user_name }}!"
```

## Condition

Dalam playbook Ansible, kondisi memungkinkan Anda untuk mengeksekusi tugas berdasarkan keadaan atau kondisi tertentu. Ini memungkinkan Anda untuk membuat playbook yang lebih dinamis dan fleksibel, di mana tugas-tugas tertentu hanya dijalankan jika kondisi yang ditentukan terpenuhi. Berikut adalah beberapa detail lebih lanjut tentang kondisi dalam playbook Ansible:

### Jenis Kondisi

- **When Clause**: Kondisi ini memungkinkan Anda untuk menentukan tugas mana yang akan dieksekusi berdasarkan nilai variabel atau ekspresi boolean tertentu.
- **Loop Control**: Dengan menggunakan loop, Anda juga dapat menerapkan kondisi pada iterasi yang dilakukan oleh loop, seperti menghentikan loop saat kondisi tertentu terpenuhi.
- **Failed and Changed Conditions**: Anda dapat menentukan tindakan yang akan diambil berdasarkan apakah sebuah tugas telah gagal atau telah mengubah keadaan sistem.

### Sintaks Kondisi

- **When Clause**: Ini diterapkan pada tingkat tugas dan memiliki sintaks seperti berikut:
  ```yaml
  tasks:
    - name: Task Name
      command: some_command
      when: condition
  ```
  `condition` dapat berupa variabel, ekspresi boolean, atau perbandingan.
- **Loop Control**: Ketika menggunakan loop, kondisi dapat diterapkan di dalam loop:
  ```yaml
  - name: Task Name
    command: "{{ item }}"
    loop:
      - task1
      - task2
    when: condition
  ```

### Ekspresi Kondisi

- Kondisi dalam playbook Ansible dapat berupa ekspresi boolean, perbandingan nilai, atau evaluasi variabel.
- Anda juga dapat menggunakan fungsi-fungsi Ansible yang disediakan untuk mengevaluasi kondisi, seperti `changed_when`, `failed_when`, dll.

### Referensi Kondisi

- Anda dapat merujuk ke variabel, fakta, atau nilai kembalian dari tugas sebelumnya dalam mengevaluasi kondisi.
- Penggunaan Jinja2 sangat berguna dalam mengevaluasi dan merangkai kondisi yang kompleks.

### Kondisi Khusus

- Ansible menyediakan kondisi khusus seperti `changed`, `failed`, `skipped`, `succeeded`, dan lain-lain yang dapat Anda gunakan untuk mengeksekusi tugas berdasarkan hasil dari tugas sebelumnya.

### Example
1. 

```yaml
# Conditional example
- name: Check if variable meets condition
  hosts: localhost
  gather_facts: no
  vars:
    my_var: 10
  tasks:
    - name: Print message if variable meets condition
      debug:
        msg: "Variable meets condition: {{ my_var }}"
      when: my_var > 5
```

2. 
```yaml
# Conditional loop example
- name: Check and print items meeting condition
  hosts: localhost
  gather_facts: no
  vars:
    my_list:
      - 3
      - 8
      - 12
      - 5
  tasks:
    - name: Print items greater than 5
      debug:
        msg: "Item {{ item }} is greater than 5"
      loop: "{{ my_list }}"
      when: item > 5
```

## Looping

Looping adalah salah satu fitur yang kuat dalam playbook Ansible yang memungkinkan Anda untuk melakukan iterasi atau pengulangan tugas pada satu atau lebih item. Ini sangat berguna ketika Anda perlu melakukan operasi yang sama pada beberapa host atau ketika Anda perlu melakukan serangkaian tindakan pada satu host.

### Jenis Looping

Ada beberapa jenis looping yang bisa Anda gunakan dalam playbook Ansible:

1. **Dengan Directive `loop`**: Anda dapat menggunakan directive `loop` untuk melakukan iterasi pada daftar item yang ditentukan.

    Contoh penggunaan:

    ```yaml
    - name: Copy multiple files
      copy:
        src: "{{ item.src }}"
        dest: "{{ item.dest }}"
      loop:
        - { src: 'file1.txt', dest: '/path/to/destination1/' }
        - { src: 'file2.txt', dest: '/path/to/destination2/' }
    ```

2. **Dengan Modul `with_items`**: Sebelum Ansible versi 2.5, looping dilakukan dengan modul `with_items`. Walaupun kini lebih disarankan menggunakan `loop`, namun modul ini masih didukung.

    Contoh penggunaan:

    ```yaml
    - name: Install multiple packages
      yum:
        name: "{{ item }}"
        state: present
      with_items:
        - httpd
        - mariadb
        - php
    ```

3. **Dengan Hasil Sebelumnya**: Anda juga dapat melakukan looping berdasarkan hasil tugas sebelumnya menggunakan directive `with_items` atau `loop`.

    Contoh penggunaan:

    ```yaml
    - name: Retrieve list of users
      shell: "getent passwd | cut -d: -f1"
      register: users

    - name: Print users
      debug:
        msg: "User {{ item }}"
      loop: "{{ users.stdout_lines }}"
    ```

### Penggunaan Ekstensif

Looping sangat berguna dalam berbagai skenario, seperti:
- Mengulangi serangkaian tugas pada sekelompok host yang sama.
- Memproses daftar item yang diperoleh dari hasil tugas sebelumnya.
- Menjalankan perintah yang sama dengan parameter yang berbeda pada setiap iterasi.

### Example
```yaml
# Looping example
- name: Print messages using a loop
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Print messages
      debug:
        msg: "Item: {{ item }}"
      loop:
        - "Message 1"
        - "Message 2"
        - "Message 3"
```

## Handlers

Handlers adalah tindakan yang akan dieksekusi hanya jika satu atau lebih tugas dalam playbook telah menghasilkan perubahan. Mereka berguna ketika Anda ingin menangani kejadian spesifik, seperti memulai ulang layanan atau melakukan tindakan lain setelah konfigurasi tertentu telah berubah.

### Penggunaan Handlers

1. **Pemanggilan Handlers**: Anda dapat memanggil handler menggunakan modul `notify` pada tugas yang akan memicu handler.

    Contoh penggunaan:

    ```yaml
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
      notify: restart apache
    ```

2. **Definisi Handlers**: Handlers didefinisikan di bagian bawah playbook dengan menggunakan kunci `handlers`. Anda dapat mendefinisikan satu atau lebih handler.

    Contoh penggunaan:

    ```yaml
    handlers:
      - name: restart apache
        service:
          name: apache2
          state: restarted
    ```

3. **Memanggil Handlers dari Berbagai Tugas**: Anda dapat memanggil handler dari beberapa tugas yang berbeda. Ansible akan menunggu hingga semua tugas yang memicu handler selesai sebelum menjalankan handler.

### Kelebihan Penggunaan Handlers

- **Eksklusif**: Handlers hanya akan dijalankan jika tugas-tugas yang memanggilnya menghasilkan perubahan.
- **Pendeteksian Otomatis**: Ansible secara otomatis mengelola pemanggilan handler dan memastikan bahwa setiap handler hanya dijalankan sekali, bahkan jika dipanggil dari beberapa tugas yang berbeda.
- **Tanggapan Terhadap Perubahan**: Handlers berguna untuk menangani reaksi terhadap perubahan yang dihasilkan oleh tugas-tugas dalam playbook, seperti memulai ulang layanan setelah konfigurasi telah diubah.

### Penggunaan Ekstensif

Handlers sangat berguna dalam berbagai skenario, seperti:

- Memulai ulang layanan setelah konfigurasi telah diubah.
- Menjalankan tugas pemeliharaan setelah pembaruan perangkat lunak.
- Mengirim notifikasi setelah tugas selesai dieksekusi.

Dengan pemahaman tentang konsep handlers ini, Anda dapat membuat playbook Ansible yang lebih responsif dan dapat diandalkan untuk mengelola infrastruktur Anda dengan lebih baik.

### Example

```yaml
---
- hosts: localhost
  tasks:
    - name: Perform some task that may trigger the handler
      command: echo "Task executed"
      changed_when: true
      notify: debug message

  handlers:
    - name: debug message
      debug:
        msg: "This is a debug message triggered by the handler"
```


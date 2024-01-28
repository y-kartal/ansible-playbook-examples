# Ders Notları

  - Terraformu ne zaman ansible ne zaman kullanırız
    - Terraform ile makineleri olusturyoruz ansible ile makineleri configure ediyoruz
  
  - Ansibel ethoc komutları:
    - komut bır kere calısır gerıye donuk takıp edilmez
    - acil durumlarda kullanılır

  - Tercih edilen declaretive yöntemdir

  - Ansibilde bir tane kontrol node olmazk zorunda 
    - Bu makinede ansible olmak zorunda 
    - İnventery file olmak zorunda 
      - hangi pc leri kontrol edilceğimiz soyledıgımız file
      - İnventry dosyasının nerede oldugunu config dosyasına belirtiyoruz
      - İnventery ansible config altında belirtiyoruz
    
    - Aclıntes:
      - Control ettiğimiz makinelerde ansible olmasına gerek yok
      - Control ettiğimiz makinelerde python yüklü olsa yeterli

    - Ansible.cfg:
      - Default olarak var istersek kendimizde üretebiliriz

      - Windowsa winrm ile baglanır
      - Host_key_checking = False
        - paket yuklerken sormasın dıye
      - interpreter-python=auto_silent
        - python version uyarısı vermesın dıye

    - Ansible olmazsa olmazımız
      - İnventr dosyası olmalı
      - Ansible yüklü control node olmalı 
      - Ansible.cfg dosyası ve öncelik sırası
              - ANSIBLE_CONFİG env
              - Bulunduğu dizin
              - ~/.ansible.cfg (kullanıcı ana dizininde)
              - /etc/ansible/ansible.cfg
              - hicbi yerde yoksa 
      - CLI her zaman onceliklidir

      - Playbook
        - Play
          - Task
            - Modul
              - Argumanlar

    - How to run
      - ansible-playbook .yml

    - Host and Users
      - Host işlem yapacagımız serverler
      - User da hangi kullanı ile işlem yapıcaz

    - İnventery File
      - Yöneticeğimiz server/makinlerinin bilgilerini girdğimiz yer
      - default yeri /etc/ansible/hosts.  altında 
        - son kurulumlarda bizden olusturmamızı istiyor

    - Task:
      - Her bir tesk bir modul kullanır
      - Bir taskin altında birden fazla modul olabilir

    - Modules:
      - İçerisinde python kodu olan programlar

    - İdempotetn özelliği:
      - Yaptıgı ısı tekrar etmıyor
      - shelde bu ozellik yok, her calıstıgımızda işlem yapılmamıs gıbı tekrar eder
    
    - Handlers:
      - Calısması baska bir taske baglıdır

    - Variables:
      - Playbook içinde de vars baglıgı altında yapabılırız
      - Var.file diyip dosyadan da alabiliriz
      - inventeriy dosyasında ve playbook içinde var varsa p

    - Conditionals:
      - when altında tasklerin calsıması ıcın sart kosuyoruz


    - Loop:
      - hangi paketleri yükyeceğimizi yazıyoruz



# Hands-on Ansible-02 : Using Playbook with Tasks

Asagıdakı  Makineleri kurduk 
1-control-node
2-node1
3-node2
4-node3 (ubuntu)


- To confirm the successful installation of Ansible. You can run the following command.

```bash
ansible --version

  config file = /home/ec2-user/lesson/ansible.cfg
    - config dosyasının yolu 

  configured module search path = ['/home/ec2-user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
    - 

  ansible python module location = /usr/lib/python3.9/site-packages/ansible
    - python dosyanın yolu


```
### Configure Ansible on AWS EC2

- inventory dosyanı terraform olsutugu ıcın asagıdakılere gerek yok

```bash
inventory.txt
```
```bash
[webservers]
node1 ansible_host=<node1_ip> ansible_user=ec2-user
node2 ansible_host=<node2_ip> ansible_user=ec2-user

[ubuntuservers]
node3 ansible_host=<node3_ip> ansible_user=ubuntu

[all:vars]
ansible_ssh_private_key_file=/home/ec2-user/<pem file>
```

- Create an  `ansible.cfg` file under `/home/ec2-user` folder as below. 

```bash
vim ansible.cfg
[defaults]
host_key_checking = False
inventory = inventory.txt
deprecation_warnings=False
interpreter_python=auto_silent
```

- Copy your pem file to the `/home/ec2-user` directory. First go to your pem file directory on your local computer and run the following command.

```bash
scp -i <pem file> <pem file> ec2-user@<public DNS name of the control node>:/home/ec2-user
  - pem dosyanın kopyalanması
```

- or you can create a file name <pem file> into the directory `/home/ec2-user` on the control node and copy your pem file into it.

## Part 2 - Ansible Playbooks

- ansible all -m ping
  - komutu ile diger makinelere baglantı kontrolu yapıyoruz


- Create a yaml file named "playbook1.yml" and make sure all our hosts are up and running.

```yml

  - yml dosyası ıle inventory ve ansible.cfg aynı yerde olmalı
  - calısırken calıstıgı dızıne bakıyor

---
- name: Test Connectivity
    # play ismi hata ayıklaması yaparken önemli
  hosts: all
    # hangi makinelerde yapsın "*" yaparsak da aynı anlama gelir  
  tasks:
   - name: Ping test
        # taske isim yazıyoruz
     ansible.builtin.ping:
        # ping modulunu kullanıyoruz
```

- Run the yaml file.

```bash
ansible-playbook playbook1.yml

PLAY [Test Connectivity] ****************************************************************************************************************************************************************************************************
  - Taske basladım dıyor

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************************
  - asagıda yazan makine bilgilerini topluyor

ok: [node3]
ok: [node1]
ok: [node2]

TASK [Ping test] ************************************************************************************************************************************************************************************************************
  - asagıda belirtilen nodelarde ping modulunu calsıtırdı

ok: [node3]
ok: [node1]
ok: [node2]

PLAY RECAP ******************************************************************************************************************************************************************************************************************
  - yaptıgı işlemlerin özeti 

node1                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node2                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node3                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  

```

- Create a text file named "testfile1" and write "Hello Clarusway" with using vim. Then create a yaml file name "playbook2.yml" and send the "testfile1" to the hosts. 

```yml
---
- name: Copy for linux
    # play ismi hata ayıklaması yaparken önemli
  hosts: webservers
    # webservers grubunda calıstırıyoruz
  tasks:
   - name: Copy your file to the webservers
        # taske isim yazıyoruz
     ansible.builtin.copy:
        # taske isim yazıyoruz
       src: /home/ec2-user/testfile1
         # coppy modulunu kullanarak testfile1 bu dosya yolundan kopyalıyoruz   
       dest: /home/ec2-user/testfile1
         # coppy modulunu kullanarak file buraya kopyalıyoruz ısmı degıstırebılırız

- name: Copy for ubuntu
  hosts: ubuntuservers
  tasks:
   - name: Copy your file to the ubuntuservers
     ansible.builtin.copy:
       src: /home/ec2-user/testfile1
       dest: /home/ubuntu/testfile1
```

- Run the yaml file.

```bash
ansible-playbook playbook2.yml

- Dosyaların gidip gitmedıgını control etmek ıcın
  - ansible all -m shell -a "ls"
    - -m shell: Bu, shell modülünün kullanılacağını belirtir. 

    - -a "ls": Bu, shell komutunu belirtir. 

  -  ansible all -m shell -a "ls ; cat test*"
      - içerisne yazılanları kontrol emek için

```

- Connect the nodes with SSH and check if the text files are copied or not. 

- Create a yaml file named playbook3.yml as below.

```bash
vim playbook3.yml
```
```yml
- name: Copy for linux
  hosts: webservers
  tasks:
   - name: Copy your file to the webservers
     ansible.builtin.copy:
       src: /home/ec2-user/testfile1
       dest: /home/ec2-user/testfile1
       mode: u+rw,g-wx,o-rwx
        - copyaladıgımız file modunu duzenlıyoruz


- name: Copy for ubuntu
  hosts: ubuntuservers
  tasks:
   - name: Copy your file to the ubuntuservers
     ansible.builtin.copy:
       src: /home/ec2-user/testfile1
       dest: /home/ubuntu/testfile1
       mode: u+rw,g-wx,o-rwx

- name: Copy for node1
  hosts: node1
  tasks:
   - name: Copy using inline content
     ansible.builtin.copy:
       content: 'This is content of file2'
          # olusacak dosyanın ıcerısınde n yazmak ıstersek onu yazıyoruz
       dest: /home/ec2-user/testfile2
          # kopyalanacak yer

   - name: Create a new text file
     ansible.builtin.shell: "echo Hello World > /home/ec2-user/testfile3"
        # shell modulunu kullanarak yukarıdakı işlemi yaptık
```

- Run the yaml file.

```bash
ansible-playbook playbook3.yml
```
- Connect the node1 with SSH and check if the text files are there.

- Install Apache server with "playbook4.yml". After the installation, check if the Apache server is reachable from the browser.

```bash
ansible-doc yum
  - doc olarak görmek ıstersek
ansible-doc apt

vim playbook4.yml
```
```yml
---
- name: Apache installation for webservers
  hosts: webservers
  become: true
    # dersek sudo olur
  tasks:
   - name: install the latest version of Apache
     ansible.builtin.yum:
        - node1 ve node2 makinelere redhat tabanlı oldugu ıcın yum modulunu kullanıyoruz
       name: httpd
        - apachenın yum paketındekı karsılıgı
       state: latest
        - son surum
        - https://phoenixnap.com/kb/install-apache-on-centos-7 
        - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html

   - name: start Apache
     ansible.builtin.shell: "service httpd start"
      - apache serveri baslattık

- name: Apache installation for ubuntuservers
  hosts: ubuntuservers
  tasks:
   - name: update
     ansible.builtin.shell: "apt update -y"
     
   - name: install the latest version of Apache
     ansible.builtin.apt:
       name: apache2
       state: latest
        - present: kurulu degılse latest versıyonu kurar, bi kere kurunca varsa bır daha kurmuyor versıon degısıklıgını dıkkate almıyor

  sudo yekısı ıster 
  ansible-playbook -b playbook4.yml
  -b onun ıcın kullandık
  - kontrol icin consoldan control edebiriz
```
- Run the yaml file.

```bash
ansible-playbook -b playbook4.yml
ansible-playbook -b playbook4.yml   # Run the command again and show the changing parts of the output.
```

- Create playbook5.yml and remove the Apache server from the hosts.

```bash
vim playbook5.yml
```
```yml
---
- name: Remove Apache from webservers
  hosts: webservers
  tasks:
   - name: Remove Apache
     ansible.builtin.yum:
       name: httpd
       state: absent
        # silme ıcın remove de yazabiliriz
       autoremove: yes
        # gereksız paketleri de silmek için

- name: Remove Apache from ubuntuservers
  hosts: ubuntuservers
  tasks:
   - name: Remove Apache
     ansible.builtin.apt:
       name: apache2
       state: absent
       autoremove: yes
       purge: yes
```

- Run the yaml file.

```bash
ansible-playbook -b playbook5.yml
```

- This time, install Apache and wget together with playbook6.yml. After the installation, enter the IP-address of node2 to the browser and show the Apache server. Then, connect node1 with SSH and check if "wget and apache server" are running. 

```bash
vim playbook6.yml
```
```yml
---
- name: Apache installation and configuration for ubuntuservers
  hosts: ubuntuservers
  tasks:
   - name: installing apache
     ansible.builtin.apt:
       name: apache2
       state: latest

   - name: index.html
     ansible.builtin.copy:
       content: "<h1>Hello DOSTUM</h1>"
       dest: /var/www/html/index.html

   - name: restart apache2
     ansible.builtin.service:
        - modelu ile restart ettık
       name: apache2
       state: restarted
       enabled: yes

- name: Apache and wget installation for webservers
  hosts: webservers
  tasks:
    - name: installing httpd and wget
      ansible.builtin.yum:
        pkg: "{{ item }}"
          - isimlerin yerine kullandık item diyince paket yukleyecegını anlıyor
        state: present
          - present diyince kurulu varmı ona bakar yoksa son surumu yukler kurlu ıse atlar
      loop:
        - httpd
        - wget
```

- Run the yaml file.

```bash
ansible-playbook -b playbook6.yml
```

- Remove Apache and wget from the hosts with playbook7.yml.

```bash
vim playbook7.yml
```
```yml
---
- name: Remove Apache from ubuntuservers
  hosts: ubuntuservers
  tasks:
   - name: Uninstalling Apache
     ansible.builtin.apt:
       name: apache2
       state: absent
       update_cache: yes
        - güncel paket verilerine erişip cache siliyor 
       autoremove: yes
        - artık kullanılmayan veya başka bir pakete bağımlı olmayan paketleri otomatik olarak kaldırır.
       purge: yes
        - pakete bağlı olan tüm yapılandırma dosyalarını da kaldırır.

- name: Remove Apache and wget from webservers
  hosts: webservers
  tasks:
   - name: removing apache and wget
     ansible.builtin.yum:
       pkg: "{{ item }}"
       state: absent
     loop:
       - httpd
       - wget

yüklendimi kontrol için
  - ansibe-playbook -m shell -a "wget --version"
```

- Run the yaml file.

```bash
ansible-playbook -b playbook7.yml
```

- Using ansible loop and conditional, create users with playbook8.yml. 

```bash
vi playbook8.yml
```

```yml
---
- name: Create users
  hosts: "*"
    - butun hostlarda kullancak
  tasks:
    - name: Create user for REDHAT OS FAMILY
      ansible.builtin.user:
        - kullanıcı olusturuyoruz
        name: "{{ item }}"
          - loopla yapmak ıcın
        state: present
          - state: Bu, kullanıcının var olup olmadığını belirtir. present, kullanıcının var olması gerektiğini belirtir. absent ise kullanıcının var olmaması gerektiğini belirtir
      loop:
        - joe
        - matt
        - james
        - oliver
      when: ansible_os_family == "RedHat"
        - setup modulunde kontrol ediyor eger redhat ıse calıstırıcak

    - name: Create user for SUSE OS FAMILY
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      loop:
        - david
        - tyler
      when: ansible_os_family == "SUSE"

    - name: Create user for DEBIAN OS FAMILY
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      loop:
        - john
        - aaron
      when: ansible_os_family == "Debian" or ansible_distribution_version == "20.04"
```

- Run the playbook8.yml

```bash
ansible-playbook -b playbook8.yml
```
- ansible all -m setup | grep ansible_os_family
  - tüm makinlerin işletim sistemini cekmek ıcın

- ansible all -m setup | grep ansible_distribution_version
  - ansible_distribution_version kontrol etmek ıstersek

- ansible all -m shell -a "sudo cat /etc/passwd | tail -n5"
  - son 5 kullancıyı gormek ısterse

- ansible all -m shell -a "cd .. ; ls"
  - home folderının altında kullancıları olusturgu ıcın kullanılara bu komutla bakıyoruz
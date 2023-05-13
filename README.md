# Bash
Для выполнения этого действия требуется установить приложением git:
`git clone https://github.com/altyn-kenzhebaev/ansible-hw11.git`
В текущей директории появится папка с именем репозитория. В данном случае ansible-hw11. Ознакомимся с содержимым:
```
cd ansible-hw11
ls -l
ansible.cfg 
hosts
nginx.yml
README.md
roles
Vagrantfile
```
Здесь:
- README.md - файл с данным руководством
- Vagrantfile - файл описывающий виртуальную инфраструктуру для `Vagrant`
- nginx.yml - главный файл запуска ansible-playbook
- hosts - инвентаризационный файл с хостами
- ansible.cfg - конфиг-файл ansible
- roles - папка, где размещены файлы роли nginx
Запускаем ВМ:
```
vagrant up
```
# nginx.yml 
Файл ansible-playbook с запуском роли
```
---
- name: NGINX | Install and configure NGINX
  hosts: nginx
  become: true

  roles:
    - nginx    
...
```
# Разбор роли:
Структура роли
```
tree
.
├── ansible.cfg
├── hosts
├── nginx.yml
├── README.md
├── roles
│   └── nginx
│       ├── handlers
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   └── nginx.conf.j2
│       └── vars
│           └── main.yml
```
handlers/main.yml:
```
- name: restart nginx
  systemd:
    name: nginx
    state: restarted
    enabled: yes

- name: reload nginx
  systemd:
    name: nginx
    state: reloaded
```
tasks/main.yml:
```
- name: NGINX | Install EPEL Repo package from standart repo
  yum:
    name: epel-release
    state: present
  tags:
    - epel-package
    - packages

- name: NGINX | Install NGINX package from EPEL Repo
  yum:
    name: nginx
    state: latest
  notify:
    - restart nginx
  tags:
    - nginx-package
    - packages

- name: NGINX | Create NGINX config file from template
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - reload nginx
  tags:
    - nginx-configuration
```
templates/nginx.conf.j2:
```
# {{ ansible_managed }}
events {
    worker_connections 1024;
}

http {
    server {
        listen       {{ nginx_listen_port }} default_server;
        server_name  default_server;
        root         /usr/share/nginx/html;

        location / {
        }
    }
}
```
vars/main.yml:
```
nginx_listen_port: 8080
```
Запуск playbook:
```
ansible-playbook nginx.yml 

PLAY [NGINX | Install and configure NGINX] ***************************************************

TASK [Gathering Facts] ***********************************************************************
ok: [nginx]

TASK [nginx : NGINX | Install EPEL Repo package from standart repo] **************************
changed: [nginx]

TASK [nginx : NGINX | Install NGINX package from EPEL Repo] **********************************
changed: [nginx]

TASK [nginx : NGINX | Create NGINX config file from template] ********************************
changed: [nginx]

RUNNING HANDLER [nginx : restart nginx] ******************************************************
changed: [nginx]

RUNNING HANDLER [nginx : reload nginx] *******************************************************
changed: [nginx]

PLAY RECAP ***********************************************************************************
nginx                      : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
Проверка:
```
~/O/ansible-hw11$ nc -zv 192.168.50.150 8080
Connection to 192.168.50.150 8080 port [tcp/http-alt] succeeded!
```
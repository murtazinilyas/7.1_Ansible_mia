# Домашнее задание к занятию «Ansible.Часть 2» Муртазин Ильяс fops-42

### Задание 1

**Выполните действия, приложите файлы с плейбуками и вывод выполнения.**

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны: 

1. Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать [официальный сайт](https://kafka.apache.org/downloads) и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
2. Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
3. Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

### Решение 1

*Плейбук 1:*

```YAML
- hosts: "my"
  become: true
  tasks:
    - name: "Create directory"
      file:
        path: "{{ ansible_user_dir }}/apache_mia"
        state: directory
    - name: "Unpack archive"
      unarchive:
        src: "https://dlcdn.apache.org/kafka/4.1.0/kafka-4.1.0-src.tgz"
        dest: "{{ ansible_user_dir }}/apache_mia"
        remote_src: yes
```

*Результаты выполнения плейбука:*
![Result t1-1](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-1_res.png)
![t1-1 on net1](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-1_net1.png)
![t1-1 on net2](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-1_net2.png)

*Плейбук 2:*
```YAML
---
- hosts: "my"
  become: true
  tasks:
  - name: "Install tuned"
    apt:
      name: tuned
      state: present
      update_cache: yes
  - name: "Start tuned"
    systemd:
      name: tuned
      enabled: true
      masked: no
```

*Результаты выполнения плейбука:*
![Result t1-2](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-2_res.png)
![t1-2 on net1](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-2_net1.png)
![t1-2 on net2](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-2_net2.png)

*Плейбук 3:*

```YAML
---
- hosts: "my"
  become: true
  vars:
    hello: "Hello world!"
  tasks:
  - name: "Change /etc/motd"
    copy:
      content: "{{ hello }}"
      dest: /etc/motd
```

*Результаты выполнения плейбука:*
![Result t1-3](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-3_res.png)
![t1-3 on net1](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-3_net1.png)
![t1-3 on net2](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t1-3_net2.png)

---

### Задание 2

**Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.** 

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору. 

### Решение 2

*Модифицированные плейбук 3:*

```YAML
---
- hosts: "my"
  become: true
  vars:
    hello: "Hello admin! Good luck and have a nice day to you!"
  tasks:
  - name: "Change /etc/motd"
    copy:
      content:
        - "{{ hello }}"
        - "{{ ansible_facts.default_ipv4.address }}"
        - "{{ ansible_facts.hostname }}"
      dest: /etc/motd
```

*Результаты выполнения плейбука:*
![Result t2](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t2_res.png)
![t2 on net1](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t2_net1.png)
![t2 on net2](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t2_net2.png)

---

### Задание 3

**Выполните действия, приложите архив с ролью и вывод выполнения.**

Ознакомьтесь со статьёй [«Ansible - это вам не bash»](https://habr.com/ru/post/494738/), сделайте соответствующие выводы и не используйте модули **shell** или **command** при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

1. Установить веб-сервер Apache на управляемые хосты.
2. Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес.
Используйте [Ansible facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html) и [jinja2-template](https://linuxways.net/centos/how-to-use-the-jinja2-template-in-ansible/). Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
4. Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
5. Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

В качестве решения:
- предоставьте плейбук, использующий роль;
- разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
- предоставьте скриншоты выполнения плейбука;
- предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.

### Решение 3

*Плейбук, использующий роль:*

```YAML
---
- hosts: "my"
  become: true
  roles:
    - mia_apache
```
Ссылка на архив с используемой ролью [mia_apache.tar.gz](https://drive.google.com/file/d/12SwuJfvy9v-x_tlMFKkT30oZKML0SYo6/view?usp=sharing)

**/mia_apache/tasks/main.yml**
```YAML
---
- name: "Install Apache server"
  apt:
    name: apache2
    state: present
    update_cache: yes
- name: "Start apache"
  service:
    name: apache2
    state: started
- name: "Configure index.html"
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
- name: "Open 80 port"
  ufw:
    rule: allow
    port: 80
    proto: tcp
- name: "Checking availability of server"
  uri:
    url: http://localhost:80
    method: GET
    status_code: 200
    return_content: yes
```

**/mia_apache/handlers/main.yml**
```YAML
---
- name: "Restart apache server"
  service:
    name: apache2
    state: restarted
```

**/mia_apache/defaults/main.yml**
```YAML
---
cpu: "{{ ansible_processor }}"
ram: "{{ ansible_memtotal_mb }}"
hdd: "{{ ansible_facts['devices']['sda']['size'] }}"
ip4: "{{ ansible_enp0s8['ipv4']['address'] }}"
```

**/mia_apache/templates/index.html.j2**
```html
!DOCTYPE html>
<html>
<body>

<h1>System info:</h1>

<p>CPU info: {{ cpu }}</p>
<p>Total RAM: {{ ram }}</p>
<p>Capacity of first HDD: {{ hdd }}</p>
<p>Host ipv4 address: {{ ip4 }}</p>

</body>
</html>
```

*Результат выполнения плейбука:*
![Result t3](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t-3_res.png)

*Модифицированные стартовые страницы сервера Apache:*
![Index.html on net1 and net2](https://github.com/murtazinilyas/7.1_Ansible_mia/blob/main/scshots/a_t3_ind.png)

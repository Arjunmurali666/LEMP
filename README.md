# LEMP
## Ansible - This is an ansible playbook to install WordPress on LEMP environment.

```ansible
---
- name: 'installing wordpress'
  hosts: vipoint
  become: yes
  vars:
    domain: example.com
    mysql_root: mysqlroot123
    mysql_database: wodpress
    mysql_user: wpuser
    mysql_password: wpuser

  tasks:
    - name: 'Lemp installation'
      yum:
        name:
            - php-mysql
            - mariadb-server
            - php
            - php-fpm
            - MySQL-python
    - name: 'Installing nginx using amazon linux extras'
      shell: amazon-linux-extras install nginx1.12

    - name: 'creating virtual host'
      template:
        src: nginx.j2
        dest: /etc/nginx/conf.d/{{domain}}.conf

    - name: 'creating document root'
      file:
        path: /var/www/{{domain}}
        state: directory

    - name: 'adding content to the document root'
      copy:
        content: '<h1> it Works...!!!</h1>'
        dest: /var/www/{{domain}}/index.html

    - name: 'Restarting services'
      service:
        name: "{{ item }}"
        state: restarted
      with_items:
        - nginx
        - mariadb

    - name: 'mariadb root password change'
      ignore_errors: true
      mysql_user:
        login_user: root
        login_password: ''
        user: root
        password: "{{mysql_root}}"
        host_all: true

    - name: 'Removing anonymous users'
      mysql_user:
        login_user: root
        login_password: "{{mysql_root}}"
        user: ''
        state: absent
        host_all: true

    - name: 'Mariadb creating new database'
      mysql_db:
        login_user: root
        login_password: "{{mysql_root}}"
        name: "{{mysql_database}}"
        state: present

    - name: 'Mariadb creating new user'
      mysql_user:
        login_user: root
        login_password: "{{mysql_root}}"
        user: "{{mysql_user}}"
        password: "{{mysql_password}}"
        state: present
        host: localhost
        priv: "{{mysql_database}}.*:ALL"

    - name: 'Downloading WordPress to /tmp'
      get_url:
        url: http://wordpress.org/latest.tar.gz
        dest: /tmp/latest.tar.gz

    - name: 'Extracting contents from wordpress'
      unarchive:
        src: /tmp/latest.tar.gz
        dest: /tmp/
        remote_src: yes

    - name: 'Copy wordpress content to document root'
      shell: "cp -r /tmp/wordpress/*  /var/www/{{domain}}"

    - name: 'Wordpress - Creating wp-config.php'
      template:
        src: wp-config.php.tmpl
        dest: /var/www/{{ domain }}/wp-config.php

    - name: 'Deleting wordpress from /tmp'
      file:
        path: "{{ item }}"
        state: absent
      with_items:
        - /tmp/wordpress.tar.gz
        - /tmp/wordpress/

    - name: "LempStack - Restarting Services"
      service:
        name: "{{ item }}"
        state: restarted
      with_items:
        - nginx
        - mariadb
```

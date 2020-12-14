# Introduction
This repo will show you how to provision the app node.js using ansible playbooks.

# Pre-Requisits
- Ansible
- AWS

# Instructions
1) ssh into your controller instance within AWS
2) Navigate to the /etc/ansible/hosts file and enter the ip's for your host A and host B
3) Navigate to your playbook folder 
4) Run playbook_app.yaml
5) Run playbook_db.yaml
6) Using your browser, type in the public ip of your app host instance
7) Check that the DB instance is running by navigating to the posts page.

# App playbook

## Updating and installing packages
```yaml
  - name: Update all packages to their latest version
    apt:
      name: "*"
      state: latest

  - name: Install a list of packages
    apt:
      pkg:
      - nginx
      - nodejs
      - npm
      - git
      state: present

  - name: Install PM2
    npm:
      name: pm2
      global: yes

```

## Editing files
```yaml
  - name: Remove node_modules (delete file)
    file:
      path: /home/ubuntu/app
      state: absent

  - name: Remove default file (delete file)
    file:
      path: /etc/nginx/sites-available/defualt
      state: absent

  - name: Copy port80.default file with owner and permissions
    copy:
      src: /home/ubuntu/environment/port80.default
      dest: /etc/nginx/sites-available/default
      #mode: '0644'
    notify:
     - Ben

  - name: Copy app with owner and permissions
    copy:
      src: /home/ubuntu/app
      dest: /home/ubuntu
      #mode: '0644'
```
## Starting the app
```yaml
  - name: Kill pm2
    shell: sudo pm2 kill
    args:
      chdir: /home/ubuntu/app

  - name: Install npm
    shell: sudo npm install
    args:
      chdir: /home/ubuntu/app

  - name: Start app.js
    shell: sudo DB_HOST=172.31.44.169 pm2 start app.js
    args:
      chdir: /home/ubuntu/app
```

# Db playbook

## Installing Mongod
```yaml
  - name: apt update
    shell: sudo apt update -y

  - name: Import the public key used by the package management system
    apt_key:
      keyserver=hkp://keyserver.ubuntu.com:80
      id=D68FA50FEA312927
      state=present

  - name: Add MongoDB repository
    apt_repository: repo='deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse' state=present

  - name: Install 3.2.20 mongod
    shell: sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20
```

## Replacing mongod.conf
```yaml
  - name: Remove original mongod.conf
    file:
      state: absent
      path: "/etc/mongod.conf"

  - name: Link config file
    shell: ln -s /home/ubuntu/db/mongod.conf /etc/mongod.conf


  - name: Copy mongod.conf file
    copy:
      src: /home/ubuntu/db/mongod.conf
      dest: /etc/mongod.conf
```

## Restarting and enabling mongodb
```yaml
  - name: Restarting mongod service
    shell: sudo systemctl restart mongod
    args:
      chdir: /home/ubuntu/

  - name: Enabling mongod service
    shell: sudo systemctl enable mongod.service --now
    args:
      chdir: /home/ubuntu
```

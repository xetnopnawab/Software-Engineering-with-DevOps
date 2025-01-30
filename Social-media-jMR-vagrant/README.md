# VPROFILE PROJECT SETUP

## Prerequisite

1. Oracle VM VirtualBox
2. Vagrant
3. Vagrant plugins
   - Install `hostmanager` plugin:
     ```bash
     $ vagrant plugin install vagrant-hostmanager
     ```
4. Git Bash or equivalent editor

## VM Setup

1. Clone the source code.
2. Navigate to the repository:
   ```bash
   $ cd <repository>
   ```
3. Switch to the local branch.
4. Navigate to the `Manual_provisioning` folder:
   ```bash
   $ cd vagrant/Manual_provisioning
   ```

Bring up the VMs:

```bash
$ vagrant up
```

**Note:** Setting up all the VMs may take time. If setup stops, rerun the command:

```bash
$ vagrant up
```

All VM hostnames and `/etc/hosts` file entries will be updated automatically.

---

## Provisioning Services

**Order of Setup:**

1. MySQL (Database Service)
2. Memcache (DB Caching Service)
3. RabbitMQ (Broker/Queue Service)
4. Tomcat (Application Service)
5. Nginx (Web Service)

### 1. MySQL Setup

1. Login to the `db01` VM:
   ```bash
   $ vagrant ssh db01
   ```
2. Verify `/etc/hosts` entries:
   ```bash
   # cat /etc/hosts
   ```
3. Update OS and install MariaDB:
   ```bash
   # dnf update -y
   # dnf install epel-release -y
   # dnf install git mariadb-server -y
   # systemctl start mariadb
   # systemctl enable mariadb
   ```
4. Secure MariaDB installation:

   ```bash
   # mysql_secure_installation
   ```

   Use `admin123` as the root password.

5. Set up the database and users:

   ```sql
   mysql> CREATE DATABASE accounts;
   mysql> GRANT ALL PRIVILEGES ON accounts.* TO 'admin'@'localhost' IDENTIFIED BY 'admin123';
   mysql> GRANT ALL PRIVILEGES ON accounts.* TO 'admin'@'%' IDENTIFIED BY 'admin123';
   mysql> FLUSH PRIVILEGES;
   ```

6. Initialize the database:

   ```bash
   # cd /tmp/
   # git clone -b local https://github.com/hkhcoder/vprofile-project.git
   # cd vprofile-project
   # mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
   ```

7. Restart and configure the firewall:
   ```bash
   # systemctl restart mariadb
   # systemctl start firewalld
   # systemctl enable firewalld
   # firewall-cmd --zone=public --add-port=3306/tcp --permanent
   # firewall-cmd --reload
   ```

### 2. Memcache Setup

1. Login to the `mc01` VM:
   ```bash
   $ vagrant ssh mc01
   ```
2. Install and configure Memcache:
   ```bash
   # dnf update -y
   # dnf install epel-release -y
   # dnf install memcached -y
   # systemctl start memcached
   # systemctl enable memcached
   # sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
   # systemctl restart memcached
   ```
3. Configure the firewall:
   ```bash
   # firewall-cmd --add-port=11211/tcp --permanent
   # firewall-cmd --reload
   ```

### 3. RabbitMQ Setup

1. Login to the `rmq01` VM:
   ```bash
   $ vagrant ssh rmq01
   ```
2. Install RabbitMQ:
   ```bash
   # dnf update -y
   # dnf install epel-release -y
   # dnf install rabbitmq-server -y
   # systemctl enable --now rabbitmq-server
   ```
3. Configure RabbitMQ:
   ```bash
   # rabbitmqctl add_user test test
   # rabbitmqctl set_user_tags test administrator
   # rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
   ```
4. Configure the firewall:
   ```bash
   # firewall-cmd --add-port=5672/tcp --permanent
   # firewall-cmd --reload
   ```

### 4. Tomcat Setup

1. Login to the `app01` VM:
   ```bash
   $ vagrant ssh app01
   ```
2. Install Java and Tomcat:
   ```bash
   # dnf update -y
   # dnf install java-17-openjdk java-17-openjdk-devel -y
   # wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
   # tar xzvf apache-tomcat-10.1.26.tar.gz -C /usr/local
   # mv /usr/local/apache-tomcat-10.1.26 /usr/local/tomcat
   ```
3. Configure and start Tomcat:
   ```bash
   # systemctl daemon-reload
   # systemctl start tomcat
   # systemctl enable tomcat
   ```
4. Configure the firewall:
   ```bash
   # firewall-cmd --add-port=8080/tcp --permanent
   # firewall-cmd --reload
   ```

### 5. Nginx Setup

1. Login to the `web01` VM:
   ```bash
   $ vagrant ssh web01
   ```
2. Install and configure Nginx:
   ```bash
   # apt update && apt upgrade -y
   # apt install nginx -y
   ```
3. Create a new Nginx configuration file:

   ```bash
   # vi /etc/nginx/sites-available/vproapp
   ```

   Add the following content:

   ```nginx
   upstream vproapp {
       server app01:8080;
   }

   server {
       listen 80;
       location / {
           proxy_pass http://vproapp;
       }
   }
   ```

4. Enable the site and restart Nginx:
   ```bash
   # ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
   # systemctl restart nginx
   ```

---

## Final Steps

Verify all services are running properly, and the application is accessible via the configured Nginx endpoint.

# Prerequisites

#

- JDK 11
- Maven 3
- MySQL 8

# Technologies

- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- Tomcat
- MySQL
- Memcached
- Rabbitmq
- ElasticSearch

# Database

Here,we used Mysql DB
sql dump file:

- /src/main/resources/db_backup.sql
- db_backup.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < db_backup.sql

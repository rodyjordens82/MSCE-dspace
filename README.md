# Installation Guide for DSpace Frontend and Backend

This document provides a step-by-step guide for installing the DSpace 8 frontend and backend on your local server.

## DSpace Frontend Installation

### Step 1: Update and Upgrade the System
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### Step 2: Install Node through NVM
1. Visit the NVM repository: [NVM GitHub Releases](https://github.com/creationix/nvm/releases)
2. Install NVM and Node.js:
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
   source ~/.bashrc
   nvm list-remote
   nvm install v18.20.4
   ```

### Step 3: Install Yarn
```bash
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
sudo apt update && sudo apt install yarn
yarn --version
```

### Step 4: Install the DSpace Frontend
```bash
wget -c https://github.com/DSpace/dspace-angular/archive/refs/tags/dspace-8.0.tar.gz
tar zxvf dspace-8.0.tar.gz
mv dspace-angular-dspace-8.0 dspace-8-angular
cd dspace-8-angular
yarn install
exit
```

### Step 5: Configure DSpace User and Directory
```bash
sudo su
sudo useradd -m dspace
sudo passwd dspace
sudo mkdir /dspace
sudo chown dspace:dspace /dspace
```

### Step 6: Start Yarn in `/opt`
```bash
sudo cp -r dspace-8-angular /opt/
sudo chown dspace:dspace -R /opt/dspace-8-angular
cd /opt/dspace-8-angular
yarn start
```

### Step 7: Open Frontend in Browser
Visit the DSpace frontend locally:
```
http://127.0.0.1:4000
```

### Step 8: Connect Frontend to Backend
```bash
sudo gedit /opt/dspace-8-angular/config/config.yml
```

Add the following configuration:
```yaml
rest:
  ssl: false
  host: localhost
  port: 8080
  nameSpace: /server
```

---

## DSpace Backend Installation

### Step 1: Update System
```bash
sudo apt update
sudo apt upgrade
```

### Step 2: Install Git, Java, and Ant
```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update; sudo apt install git
git --version

sudo apt install openjdk-17-jdk ant
java --version
ant --version
```

### Step 3: Install Maven
```bash
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
sudo tar xf apache-maven-3.9.9-bin.tar.gz -C /opt
sudo ln -s /opt/apache-maven-3.9.9 /opt/maven
```

Configure Maven in `.bashrc`:
```bash
echo "export M2_HOME=/opt/maven" >> ~/.bashrc
echo "export PATH=${M2_HOME}/bin:${PATH}" >> ~/.bashrc
source ~/.bashrc
mvn -version
```

### Step 4: Install PostgreSQL
```bash
wget -qO - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update
sudo apt-get install postgresql postgresql-client postgresql-contrib libpostgresql-jdbc-java -y
psql -V
```

Configure PostgreSQL:
```bash
sudo su postgres
createuser --username=postgres --no-superuser --pwprompt dspace
createdb --username=postgres --owner=dspace --encoding=UNICODE -T template0 dspace
psql --username=postgres dspace -c "CREATE EXTENSION pgcrypto;"
exit
```

Edit PostgreSQL configuration:
```bash
sudo gedit /etc/postgresql/16/main/pg_hba.conf
```

Change the `pg_hba.conf` file:
```
local all dspace md5
```

Restart PostgreSQL:
```bash
sudo /etc/init.d/postgresql restart
```

### Step 5: Configure DSpace User and Directory
```bash
sudo useradd -m dspace
sudo passwd dspace
sudo mkdir /dspace
sudo chown dspace /dspace
sudo mkdir /build
sudo chmod -R 777 /build
```

### Step 6: Install Solr
```bash
sudo mkdir /opt/solr-8.11
cd /opt/solr-8.11
sudo chown -R dspace:dspace /opt/solr-8.11
wget -c https://downloads.apache.org/lucene/solr/8.11.3/solr-8.11.3.tgz
tar xvf solr-8.11.3.tgz
/opt/solr-8.11/bin/solr start -force
/opt/solr-8.11/bin/solr status
```

### Step 7: Build DSpace
```bash
cd /build
wget https://github.com/DSpace/DSpace/archive/refs/tags/dspace-8.0.tar.gz
tar zxfv dspace-8.0.tar.gz
cd /build/DSpace-dspace-8.0
mvn -U package
cd dspace/target/dspace-installer
sudo ant fresh_install
```

### Step 8: Install and Configure Tomcat
```bash
wget https://downloads.apache.org/tomcat/tomcat-10/v10.1.30/bin/apache-tomcat-10.1.30.tar.gz
tar -xvzf apache-tomcat-10.1.30.tar.gz
sudo mv apache-tomcat-10.1.30 /opt/tomcat
```

Set environment variables:
```bash
sudo gedit /etc/profile
```

Add the following:
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export CATALINA_HOME=/opt/tomcat
```

Set up the Tomcat service:
```bash
sudo gedit /etc/init.d/tomcat
```

Add the following content to the script:

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          tomcat
# Required-Start:    $network
# Required-Stop:     $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start/Stop Tomcat server
### END INIT INFO

start() {
    sh /opt/tomcat/bin/startup.sh
}

stop() {
    sh /opt/tomcat/bin/shutdown.sh
}

case $1 in
    start|stop) $1;;
    restart) stop; start;;
    *) echo "Run as $0 <start|stop|restart>"; exit 1;;
esac
```

Make the script executable and restart services:
```bash
sudo chmod +x /etc/init.d/tomcat
sudo update-rc.d tomcat defaults
sudo service tomcat start
systemctl daemon-reload
sudo systemctl restart tomcat.service
```

### Step 9: Finalize Installation
```bash
sudo /dspace/bin/dspace create-administrator
```

Restart services:
```bash
sudo systemctl restart postgresql.service
/opt/solr-8.11/bin/solr stop -force
/opt/solr-8.11/bin/solr start -force
sudo systemctl restart tomcat.service
```

Access DSpace:
```
http://localhost:8080/server
```


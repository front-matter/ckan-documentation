# Installation

!!! Note

    The currently supported CKAN version is CKAN 2.10.1

## Hardware requirements

For a small to medium CKAN implementation, it is recommended the frontend (Web) and backend (db and solr) run on separate machines (Physical or VM's). The following is suggested as a minimum for each machine:

* 2 CPU cores
* 4 GB of RAM
* 60 GB of disk space

If you wish to run both frontend and backend on the same machine, you will almost certainly want to use 4(+) CPU cores.

## Installation options

Installing from package is the quickest and easiest way to install CKAN, but it requires Ubuntu 20.04 64-bit or Ubuntu 22.04 64-bit.

You should install CKAN from package if:

* You want to install CKAN on an Ubuntu 20.04 or 22.04, 64-bit server, and
* You only want to run one CKAN website per server

Other installation options are installation from source and installation using Docker.

At the end of the installation process you will end up with two running web applications, CKAN itself and the DataPusher, a separate service for automatically
importing data to CKAN's :doc:`/maintaining/datastore`. Additionally, there will be a process running the worker for running :doc:`/maintaining/background-tasks`. All these processes will be managed by `Supervisor <https://supervisord.org/>`_.

For Python 3 installations, the minimum Python version required is 3.7.

Host ports requirements:

    +------------+------------+-----------+
    | Service    | Port       | Used for  |
    +============+============+===========+
    | NGINX      | 80         | Proxy     |
    +------------+------------+-----------+
    | uWSGI      | 8080       | Web Server|
    +------------+------------+-----------+
    | uWSGI      | 8800       | DataPusher|
    +------------+------------+-----------+
    | Solr       | 8983       | Search    |
    +------------+------------+-----------+
    | PostgreSQL | 5432       | Database  |
    +------------+------------+-----------+
    | Redis      | 6379       | Search    |
    +------------+------------+-----------+

## Package installion

Update Ubuntu's package index.
```bash
sudo apt update
```

Install the Ubuntu packages that CKAN requires (and `git`, to enable you to install CKAN extensions).

```bash
sudo apt install -y libpq5 redis-server nginx supervisor git
```

Download the CKAN package.

```bash
wget https://packaging.ckan.org/python-ckan_2.10-jammy_amd64.deb
```

Install the CKAN package.

```bash
sudo dpkg -i python-ckan_2.10-jammy_amd64.deb
```

## Install and configure PostgreSQL

!!! Tip
    You can install PostgreSQL and CKAN on different servers. Just change the sqlalchemy.url setting in your `/etc/ckan/default/ckan.ini` file to reference your PostgreSQL server.
  
  Install PostgreSQL required packages.

```bash
sudo apt install -y postgresql
```

Next you’ll need to create a database user if one doesn’t already exist. Create a new PostgreSQL user called ckan_default, and enter a password for the user when prompted. You’ll need this password later.

```bash
sudo -u postgres createuser -S -D -R -P ckan_default
```

Create a new PostgreSQL database, called ckan_default, owned by the database user you just created.

```bash
sudo -u postgres createdb -O ckan_default ckan_default -E utf-8
```

Edit the sqlalchemy.url option in your CKAN configuration file (`/etc/ckan/default/ckan.ini`) file and set the correct password, database and database user.

```bash
sudo nano /etc/ckan/default/ckan.ini
```

## Install and configure Solr

The following instructions have been tested in Ubuntu 22.04 and are provided as a guidance only. 

```bash
sudo apt-get install openjdk-8-jdk
```

Download the latest supported version from the Solr downloads page. CKAN supports Solr version 8.x.

```bash
wget https://downloads.apache.org/lucene/solr/8.11.2/solr-8.11.2.tgz
```

Extract the install script file to your desired location.

```bash
tar xzf solr-8.11.2.tgz solr-8.11.2/bin/install_solr_service.sh --strip-components=2
```

Run the installation script.

```bash
sudo bash ./install_solr_service.sh solr-8.11.2.tgz
```

Create a new core for CKAN.

```bash
sudo -u solr /opt/solr/bin/solr create -c ckan
```
Replace the standard schema with the CKAN one.

```bash
sudo -u solr wget -O /var/solr/data/ckan/conf/managed-schema https://raw.githubusercontent.com/ckan/ckan/dev-v2.10/ckan/config/solr/schema.xml
```

Restart Solr.

```bash
sudo service solr restart
```

## Configure the Web Server

Configure Nginx to serve your CKAN site. Edit the Nginx config file (/etc/nginx/sites-available/ckan) that was installed by the CKAN package and insert the `server_name` variable in the `server` block.

```bash
sudo nano /etc/nginx/sites-available/ckan
```

```bash
server {
  client_max_body_size 100M;
  server_name demo.ckan.org;
  location /
...
```

Restart Nginx.

```bash
sudo service nginx restart
```

Configure the firewall to allow access to the CKAN web server via HTTP, HTTPS and SSH only.

Allow access to the CKAN web server via HTTP and HTTPS.

```bash
sudo ufw allow 'Nginx Full'
```

Allow access to the CKAN web server via SSH.

```bash
sudo ufw allow ssh
```

Enable the firewall.

```bash 
sudo ufw enable
```

```bash

```bash o

Install certbot to obtain a free SSL certificate for Nginx on Ubuntu 22.04.

```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Request a certificate for your domain. You will be prompted to enter an email address and agree to the terms of service. Certbot will automatically configure Nginx to use the certificate.

```bash
sudo certbot --nginx -d demo.ckan.org
```

## Set up a writable directory

CKAN needs a directory where it can write certain files, regardless of whether you are using the FileStore and file uploads or not (if you do want to enable file uploads, set the ckan.storage_path configuration option in the next section).

Create the directory where CKAN will be able to write files.

```bash
sudo mkdir -p /var/lib/ckan/default
```

Set the permissions of this directory. If you’re running CKAN with Nginx, then the Nginx’s user (`www-data` on Ubuntu) must have read, write and execute permissions on it.

```bash
sudo chown www-data /var/lib/ckan/default
sudo chmod u+rwx /var/lib/ckan/default
```

## Configure CKAN
Edit the CKAN configuration file (/etc/ckan/default/ckan.ini) to set up the following options.

```bash
sudo nano /etc/ckan/default/ckan.ini
```

Each CKAN site should have a unique `site_id`, for example

```bash
ckan.site_id = default
```

Provide the site’s URL. For example

```bash
ckan.site_url = http://demo.ckan.org
```

Initialize your CKAN database by running this command in a terminal.

```bash
sudo ckan db init
```

Reload the Supervisor daemon so the new processes are picked up
  
```bash
sudo supervisorctl reload
```

Restart the web server.

```bash
sudo service nginx restart
```
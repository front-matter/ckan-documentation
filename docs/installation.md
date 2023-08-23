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

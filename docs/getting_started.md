Once you’ve finished installing CKAN, this section will walk you through getting started with your new CKAN website, including creating a CKAN sysadmin user, some test data, and the basics of configuring your CKAN site.

## Activate the virtual environment

If you’re not already in your CKAN virtual environment, activate it now.

```bash
. /usr/lib/ckan/default/bin/activate
```

## Create a sysadmin user

The first thing you’ll need to do is create a sysadmin user. This user will be able to log in to the CKAN web interface and perform administrative tasks.

```bash
cd /usr/lib/ckan/default/src/ckan
```

You have to create your first CKAN sysadmin user from the command line. For example, to create a new user called `seanh` and make him a sysadmin:

```bash
ckan -c /etc/ckan/default/ckan.ini sysadmin add seanh email=seanh@localhost name=seanh
```
For a list of other command line commands for managing sysadmins, run

```bash
ckan -c /etc/ckan/default/ckan.ini sysadmin --help
```

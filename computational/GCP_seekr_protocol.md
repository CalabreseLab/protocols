# SEEKR on Google Cloud Platform

This document contains details on how to setup and maintain the google cloud instance of our seekr webapp. Part of the sections are a little lacking, but hopefully they will improve over time.

## Initial Setup

Hopefully this never has to be done again.

### Setting up an instance

* Go to [the console](console.cloud.google.com).
* In the menu, go to `Compute Engine` and `VM instances`.
* Click `Create Instance`.

Option | Value
--- | ---
Name | seekr-prod
Zone | us-east1-b
machine type | 2 vCPUs, 7.5GB mem
Boot disk | Ubuntu 16.04 LTS
Access scopes | Allow full access to all Cloud APIs
Allow HTTP traffic | [x]

* Create a new disk

Option | Value
--- | ---
Source type | None (blank disk)
Size (GB) | 200

Click `Create`

* ** Warning: ** You need to create a static IP address. I'm not 100% clear on how to do this. I think you go to `VPC Network` and click `External IP addresses`, then select `Reserve Static Address`.

Option | Value
--- | ---
Name | seekr
Region | us-east1
Attached to | seekr-prod

click `Reserve`

### Setting up SEEKR

* Log into the shell by clicking `SSH` in the `Connect` column of the vm instance of interest.
  * A new window should appear with a command line.

** Warning: ** We had some permission issues getting things set up. These notes likely aren't exact and should be used more as a guideline than a protocol at the moment.

```bash

#Install dependencies
sudo apt update
sudo apt install apache2
sudo apt install libapache2-mod-wsgi-py3
sudo apt install python3-dev
sudo apt install virtualenv

#Clone, or otherwise download seekr repository
tar -xzf seekr-master-eb4eb2030c1d792de5e128a31f7a381161361ebb.tar.gz

#Make sure apache can see the repository
cd /var/www/html/
sudo ln -sT ~/seekr-master-eb4eb2030c1d792de5e128a31f7a381161361ebb seekr

# Setup virtualenv
cd seekr
virtualenv -p python3 venv
. venv/bin/activate
pip install -r requirements.txt

# I think this gives permission for apache to execute seekr
sudo chgrp -R www-data /var/www/html/seekr/
sudo chown www-data *
vi /etc/apache2/sites-available/000-default.conf
```

Once inside `vi`, Make sure the first "paragraph" of `000-default.conf` looks like:

```
ServerAdmin webmaster@localhost
DocumentRoot /var/www/html/seekr

WSGIDaemonProcess seekr threads=5
WSGIScriptAlias / /var/www/html/seekr/app.wsgi
<Directory seekr>
       WSGIProcessGroup seekr
       WSGIApplicationGroup %{GLOBAL}
       Order deny,allow
       Allow from all
</Directory>
```

Once things have been configured, restart the apache server. I'm not sure which command works, but it's one of:

```
sudo systemctl restart apache2 #This one seems to work best
sudo apachectl restart
sudo service apache2 restart
```

## Checking Error files

Errors get logged to a couple places. Apache errors usually go to:

```
/var/log/apache2/error.log
```

But this file gets rotated and backed up, so it's worth checking the whole folder as well.
And errors from the seekr app can be found at:

```
/var/www/html/seekr/seekr_server.log
```

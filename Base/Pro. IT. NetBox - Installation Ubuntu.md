***Source***

NetBox: Installation Ubuntu (https://netbox.readthedocs.io/en/stable/installation/3-netbox/)

***Resume***

-# postgres:

sudo apt update
sudo apt install -y postgresql

/var/lib/pgsql/data/pg_hba.conf:

host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5

sudo systemctl start postgresql
sudo systemctl enable postgresql

-# check:
psql -V

sudo -u postgres psql

CREATE DATABASE netbox;
CREATE USER netbox WITH PASSWORD 'J5brHrAXFLQSif0K';
GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;

\q # to quit

-# for verify connection:
psql --username netbox --password --host localhost netbox
\conninfo

-# redis:
sudo apt install -y redis-server

-# check:
redis-server -v

-# /etc/redis.conf or /etc/redis/redis.conf

-# verify:
redis-cli ping

-# NetBox
sudo apt install -y python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev

-# Download NetBox
-# Latest version: 
sudo wget https://github.com/netbox-community/netbox/archive/vX.Y.Z.tar.gz
sudo tar -xzf vX.Y.Z.tar.gz -C /opt
sudo ln -s /opt/netbox-X.Y.Z/ /opt/netbox

-# or clone the git repo:
sudo mkdir -p /opt/netbox/
cd /opt/netbox/
sudo apt install -y git
sudo git clone -b master --depth 1 https://github.com/netbox-community/netbox.git .

sudo adduser --system --group netbox
sudo chown --recursive netbox /opt/netbox/netbox/media/

cd /opt/netbox/netbox/netbox/
sudo cp configuration.example.py configuration.py

-#config:

    ALLOWED_HOSTS
    DATABASE
    REDIS
    SECRET_KEY
	
ALLOWED_HOSTS = ['netbox.example.com', '192.0.2.123']
-# or
ALLOWED_HOSTS = ['*']

DATABASE = {
    'NAME': 'netbox',               # Database name
    'USER': 'netbox',               # PostgreSQL username
    'PASSWORD': 'J5brHrAXFLQSif0K', # PostgreSQL password
    'HOST': 'localhost',            # Database server
    'PORT': '',                     # Database port (leave blank for default)
    'CONN_MAX_AGE': 300,            # Max database connection age (seconds)
}

REDIS = {
    'tasks': {
        'HOST': 'localhost',      # Redis server
        'PORT': 6379,             # Redis port
        'PASSWORD': '',           # Redis password (optional)
        'DATABASE': 0,            # Database ID
        'SSL': False,             # Use SSL (optional)
    },
    'caching': {
        'HOST': 'localhost',
        'PORT': 6379,
        'PASSWORD': '',
        'DATABASE': 1,            # Unique ID for second database
        'SSL': False,
    }
}

python3 ../generate_secret_key.py

-# NAPALM
sudo sh -c "echo 'napalm' >> /opt/netbox/local_requirements.txt"

-# Remote File Storage
sudo sh -c "echo 'django-storages' >> /opt/netbox/local_requirements.txt"

-# Run the Upgrade Script
sudo /opt/netbox/upgrade.sh

sudo PYTHON=/usr/bin/python3.7 /opt/netbox/upgrade.sh

-# Create a Super User
source /opt/netbox/venv/bin/activate

cd /opt/netbox/netbox
python3 manage.py createsuperuser

-# Schedule the Housekeeping Task
sudo ln -s /opt/netbox/contrib/netbox-housekeeping.sh /etc/cron.daily/netbox-housekeeping

-# Test the Application
python3 manage.py runserver 0.0.0.0:8000 --insecure

Next, connect to the name or IP of the server (as defined in ALLOWED_HOSTS) on port 8000; for example, http://127.0.0.1:8000/. You should be greeted with the NetBox home page. Try logging in using the username and password specified when creating a superuser.

-# Gunicorn
sudo cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
sudo cp -v /opt/netbox/contrib/*.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl start netbox netbox-rq
sudo systemctl enable netbox netbox-rq

-# check:
systemctl status netbox.service

-# nginx
sudo apt install -y nginx
sudo cp /opt/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/netbox /etc/nginx/sites-enabled/netbox
sudo systemctl restart nginx

-# or appache
sudo apt install -y apache2
sudo cp /opt/netbox/contrib/apache.conf /etc/apache2/sites-available/netbox.conf
sudo a2enmod ssl proxy proxy_http headers
sudo a2ensite netbox
sudo systemctl restart apache2

***Tags*** #pro #it 

***Keys*** [[linkmeup]] [[NetBox]] [[REST]] [[REST API]] [[RESTfull API]] [[postgres]] [[redis]] [[nginx]] 
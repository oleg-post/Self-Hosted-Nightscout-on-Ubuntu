# VPS-Hosted-Nightscout-on-Ubuntu 18

When you obtain your Ubuntu instance from VPS provider you receive root's login and root's password. From this moment follow the procedure.

## 1. Create a New User
Once you are logged in your Ubuntu instance as root, you are prepared to add the new user account that we will use to log in from now on.

This command creates a new user called “mainuser”, but you can replace it with a username that you like:

`# adduser mainuser`

## 2. Prepare for installation

Update the Ubuntu instance:
`sudo apt-get update && sudo apt-get upgrade`

Check that Ubuntu instance has Python, npm, nodejs, git installed.

Сначала давайте проверим установлена ли у вас программа nodejs:

 `dpkg --get-selections | grep node`

Теперь вы можете ее удалить с помощью следующих команд:

 `sudo apt purge nodejs`
 

### УСТАНОВКА NODE.JS В NODE VERSION MANAGER

Чтобы установить Node js Ubuntu 18.04 с помощью NVM нам понадобится компилятор C++ в системе, а также другие инструменты для сборки. По умолчанию система не поставляется с этими программами, поэтому их необходимо установить. Для этого выполните команду:

 `sudo apt install build-essential checkinstall`

Также нам понадобится libssl:

 `sudo apt install libssl-dev`

Скачать и установить менеджер версий NVM можно с помощью следующей команды:

`$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash`

После завершения установки вам понадобится перезапустить терминал. Или можно выполнить:

`$ source /etc/profile`

Затем смотрим список доступных версий Node js:

`$ nvm ls-remote`

Дальше можно устанавливать Node js в Ubuntu, при установке обязательно указывать версию, на данный момент самая последняя 11.0, но установим десятую:

`$ nvm install 10.0`

Список установленных версий вы можете посмотреть выполнив:

`$ nvm list`

Дальше необходимо указать менеджеру какую версию нужно использовать:

 `$ nvm use 10.0`
 
Чтобы удалить эту версию node js, ее нужно деактивировать:

 `$ nvm deactivate 10.0`

Затем можно удалить:

 `$ nvm uninstall 10.0`

## Install Mongo DB
Step 1 – Setup Apt Repository

First of all, import GPK key for the MongoDB apt repository on your system using the following command. This is required to test packages before installation

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 4B7C549A058F8B6B
Lets add MongoDB APT repository url in /etc/apt/sources.list.d/mongodb.list.

Ubuntu 18.04 LTS:
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb.list
Ubuntu 16.04 LTS:
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb.list
Step 2 – Install MongoDB on Ubuntu

After adding required APT repositories, use the following commands to install MongoDB on your systems. It will also install all dependent packages required for MongoDB.

sudo apt update
sudo apt install mongodb-org
If you want to install any specific version of MongoDB, define the version number as below

sudo apt install mongodb-org=4.2.8 mongodb-org-server=4.2.8 mongodb-org-shell=4.2.8 mongodb-org-mongos=4.2.8 mongodb-org-tools=4.2.8
Step 3 – Manage MongoDB Service

After installation, MongoDB will start automatically. To start or stop MongoDB uses init script. Below are the example commands to do.

sudo systemctl enable mongod
sudo systemctl start mongod 
Use the following commands to stop or restart MongoDB service.

sudo systemctl stop mongod
sudo systemctl restart mongod 
How to Work with MongoDB – Read this tutorial
Step 5 – Verify MongoDB Installation

Finally, use the below command to check installed MongoDB version on your system.

mongod --version 

db version v4.2.8
git version: 43d25964249164d76d5e04dd6cf38f6111e21f5f
OpenSSL version: OpenSSL 1.1.1  11 Sep 2018
allocator: tcmalloc
modules: none
build environment:
    distmod: ubuntu1804
    distarch: x86_64
    target_arch: x86_64
Also, connect MongoDB using the command line and execute some test commands for checking proper working.

mongo 

> use mydb;

> db.test.save( { tecadmin: 100 } )

> db.test.find()

  { "_id" : ObjectId("52b0dc8285f8a8071cbb5daf"), "tecadmin" : 100 }


## Install CGM-Remote-Monitor (Nightscout)
Install Node.js and npm  `sudo apt-get install nodejs npm`

Download cgm-remote-monitor (nightscout) from github:
`git clone https://github.com/nightscout/cgm-remote-monitor.git`
Alternatively fork a copy of cgm-remote-monitor and clone your own copy.

`cd cgm-remote-monitor`

Install cgm-remote-monitor:
`git checkout dev`
`npm install` 

setup your cgm-remote-monitor environment as you normally would, for example creating a file my.env :
```
MONGO_CONNECTION=MONGOCONNECTIONSTRING
DISPLAY_UNITS=mmol
BASE_URL=NIGHTSCOUT_SITE_URL
DEVICESTATUS_ADVANCED="true"
mongo_collection="mogocollection_name"
API_SECRET=AVeryLongString
ENABLE=careportal%20openaps%20iob%20bwp%20cage%20basal%20pump%20bridge
BRIDGE_SERVER=EU
BRIDGE_USER_NAME=USERNAME
BRIDGE_PASSWORD=PASSWORD
```

## Install pm2 to monitor nightscout processs
`sudo npm install pm2 -g`

Start cgm-remote-monitor with pm2:
`env $(cat my.env)  PORT=1337 pm2 start server.js`

Make pm2 start cgm-remote-monitor on startup
`pm2 startup ubuntu` - this will give you a command you need to run as superuser to allow pm2 to start the app on reboot 
The command will be something like:
`sudo su -c "env PATH=$PATH:/usr/bin pm2 startup ubuntu -u username --hp /home/username"`
And then:
`pm2 save`

## Create Reverse nginx proxy

Install nginx:

`sudo apt-get install nginx`

edit this file:

`sudo vi /etc/nginx/sites-available/default `
Delete the existing contents and replace with this:
I'm assuming the proxy is on the same host as nightscout and the `proxy_pass http://127.0.0.1:1337` line - `1337` is replaced with the port that nightscout is using

```
server {
    listen 80;

    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:1337;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Then restart the nginx service 
`sudo service nginx restart`

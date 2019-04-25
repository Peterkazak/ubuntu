# JUPYTER 

## Settings as root
```
$ sudo su
cd
```
## Download and install Anaconda<br>
[Anaconda.](https://github.com/Nouvellie/ubuntu/blob/ubuntu/anaconda.md)

## Create an envs to run jupyterhub and our libraries (python3.6)
```
$ conda create --name jupyterbase python=3.6 -y
```
## Update conda
```
$ conda update -n base -c defaults conda -y
```
## Conda libs
```
$ conda install -c conda-forge mysql-connector-python -y && conda install -c pandas pymysql -y && conda install -c conda-forge mysqlclient -y && conda install -c kalefranz mysql-server -y && conda install -c conda-forge django -y && conda install -c conda-forge djangorestframework -y && conda install -c conda-forge keras -y && conda install -c anaconda tensorflow -y && conda install -c conda-forge django-cors-headers -y && conda install -c conda-forge django-filter -y && conda install -c conda-forge pandas -y && conda install -c conda-forge bokeh -y && conda install -c conda-forge appdirs -y && conda install -c conda-forge lxml -y && conda install -c conda-forge wfdb -y && conda install -c conda-forge pywavelets -y && conda install -c conda-forge sqlparse -y && conda install -c conda-forge jupyterhub -y && conda install -c conda-forge notebook -y && conda install -c conda-forge configurable-http-proxy -y && conda install -c conda-forge jupyterlab
```
## Create a jupyterhub dir (/opt/user-jupyterhub) and settings
* Install default files:
```
$ mkdir /opt/user-jupyterhub
$ conda activate jupyterbase
```
* Generate jupyterhub sqlite and cookie_secret:
```
$ jupyterhub
```
* Create a jupyterhub config: (opt/user-jupyterhub/)
```
$ jupyterhub --generate-config 
```
## Create jupyterhub cert and key: (/opt/user-jupyterhub/jupyterhub.key and /opt/user-jupyterhub/jupyterhub.crt)
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout jupyterhub.key -out jupyterhub.crt
```
## Create and add user to a group
* Create a group:
```
$ sudo groupadd groupname
```
* Add a user to a group:
```
$ sudo adduser username groupname
```
* Check group members:
```
$ sudo apt install members
$ members groupname
```
## Set jupyterhub values
* Install the jupyter lab/hub extension:
```
$ jupyter labextension install @jupyterlab/hub-extension
```
* Change the default settings:
```
$ sudo vim /opt/user-jupyterhub/jupyterhub_config.py
```
```
c.JupyterHub.port = 443
c.JupyterHub.ssl_cert = '/opt/user-jupyterhub/jupyterhub.crt'
c.JupyterHub.ssl_key = '/opt/user-jupyterhub/jupyterhub.key'
c.Spawner.cmd = ['jupyter-labhub']
c.Spawner.default_url = '/lab'
c.Spawner.notebook_dir = '~'
c.Spawner.port = 443
c.Authenticator.admin_users = ['adminuser']
c.LocalAuthenticator.group_whitelist = ['groupname']
```
* Show conda environments in jupyterhub kernell:
```
$ conda install -c conda-forge nb_conda_kernels
$ python -m ipykernel install --user --name jupyterbase --display-name "Python (jupyterbase)"

```

## Errors
* [Keyerror] User adminuser does not exist:
```
sudo rm -rf jupyterhub.sqlite
```

## As system service (systemd)
* Create service:
```
$ sudo touch /etc/systemd/system/servicename.service
```
* Configs:
```
$ sudo vim /etc/systemd/system/servicename.service
```
```
[Unit]
Description=Jupyterhub server

[Service]
Environment="PATH=/opt/anaconda3/envs/jupyterbase/bin:/opt/anaconda3/bin:/opt/anaconda3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
ExecStart=/opt/anaconda3/envs/jupyterbase/bin/jupyterhub
WorkingDirectory=/opt/user-jupyterhub
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```
## Enable server after connect
* In servicename.service:
```
[Unit]
After=syslog.target network.target
```
## Auto restart
* In servicename.service:
```
Restart=always
RestartSec=10
```
## JupyterHub config to be respected by systemd:
* In servicename.service:
```
[Service]
KillMode=process
```
* In jupyterhub_config.py:
```
c.JupyterHub.cleanup_servers = False 
```
* Reload daemon:
```
sudo systemctl daemon-reload
```
* Restart instance or service:
```
sudo systemctl enable servicename.service
```
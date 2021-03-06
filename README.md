# Multiple Subdomains Tutorial


This tutorial teaches developers how to deploy multiple apps to the same AWS EC2 instance using Nginx as a reverse proxy and how to configure subdomains. (Node and [Pylot](https://github.com/Ketul-Patel/Pylot/tree/development) apps)

---

# Instructions


## 1. First, configure at least one application for deployment. Follow the instructions below for the corresponding type of app.

###[Node Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/node_deploy.html)

###[Pylot Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/pylot_deploy.html)


---

## 2. All applications must be on a different port.


#### Node: Typically, the port is specified in the server.js file or a settings.js file using: 
```javascript
var server = app.listen(8001);
```
#### Pylot: The port must be modified in both manage.py and wsgi.py.
##### manage.py:
```python
manager.add_command('runserver', Server(host='127.0.0.1',port='5001'))
# NOTE: port must be a string.
```  

##### wsgi.py: 
```python
application.run(host='127.0.0.1',port=5001)
# NOTE: port must be an integer.
```  
---

## 3. Configure the Nginx file for all apps.


#### Overwite the existing Nginx file with this format. The first three server configs are for Node and the last two server configs are for Pylot.
  * DOMAIN.com must be replaced by the domain name
  * PROJECT#.DOMAIN.com must be replaced by the desired subdomain name
  * PORT* must be replaced by the project's port
  * PROJECT#.sock must match the configuration of the project.ini file
```
server {
  server_name DOMAIN.com;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass       http://127.0.0.1:PORT1;
    }
}
server {
  server_name PROJECT1.DOMAIN.com;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass       http://127.0.0.1:PORT2;
    }
}
server {
  server_name PROJECT2.DOMAIN.com;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass       http://127.0.0.1:PORT3;
    }
}
server {
  server_name PROJECT3.DOMAIN.com;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass       http://127.0.0.1:PORT4;
    include uwsgi_params;
    uwsgi_pass unix:/home/ubuntu/PROJECT3/PROJECT3.sock;
    }
}
server {
   server_name PROJECT4.DOMAIN.com;
   location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass         http://127.0.0.1:PORT5;
    include uwsgi_params;
    uwsgi_pass unix:/home/ubuntu/PROJECT4/PROJECT4.sock;
    }
}
```
---

## 4. Deploy other apps to the existing AWS EC2 instance.

#### Node:
```bash 
cd /var/www
sudo git clone {{project file path on github}}
cd /var/www/{{project_name}}
sudo npm install
pm2 start server.js
sudo service nginx reload && sudo service nginx restart
```
#### Pylot: 
follow instructions in highlighted blue lines via [Pylot Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/pylot_deploy.html) (except for Nginx section)
```bash 
sudo service nginx reload && sudo service nginx restart
```

---

## 5. Verify all the apps are running.
#### Node:
```bash 
pm2 status
```
#### Pylot:
```bash 
sudo start PROJECT
```

---

## 6. Go to your domain name registrar and find *Advanced DNS.*

#### Set up an "A Record" with "Host" set to @ and "Value" set to {{IP ADDRESS OF AWS INSTANCE}}
#### Set up "CNAME Record" with "Host" set to {{SUBDOMAIN}} and Value set to {{DOMAIN NAME}}

![Subdomain Configuration](https://raw.githubusercontent.com/alex-wap/subdomains/master/Subdomains.png "Subdomain Configuration")


---

## 7. Congratulations! We're done!


Note: It may take some time for subdomains to propagate (up to 24 hours). Be sure to test out the domain and subdomains.

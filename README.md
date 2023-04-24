Want to eliminate single point of failure. This can be done using a two-tier achtitecture.

Follow the normal steps to launch your ec2 instance and ssh into it. Once done,

```sudo apt install nginx -y```

You should then be able to use public ip address in webbrowers and see the nginx brand.

To copy a folder locally to your ec2 instance enter the following command into a separate terminal:

``` scp -i "tech221.pem" -r ~/Documents/tech221_test/app ubuntu@ec2-34-252-106-230.eu-west-1.compute.amazonaws.com:/home/ubuntu/  ```

The scp command first needs to know where your pem file is located, then where the folder you want to transfer is located then finally, where on the ec2 instance you want to save the file.

Here are the steps that need to be taken:

-copy app
-install required dependencies
- ensure SG allows port 3000
- npm install from the app folder location
- npm start
- app is avaiable on port 3000

Once the app folder is fully downloaded cd into the app folder and excute the provision file: ```./provision.sh```. The content of the provision file:
```
#!/bin/bash

sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install nginx -y
sudo systemctl start nginx
sudo systemctl enable
sudo apt-get install python -y
# install nodejs
sudo apt-get install python-software-properties
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install nodejs -y

# install pm2
sudo npm install pm2 -g


```
Finally, head over to aws and amend your ec2 instance security to allow traffic from port 3000.

Once done, you should be able to access the app by typing the following in a webbrowser: publicip:3000

####reverse proxy with the app

 1. make sure nginx is installed ```sudo apt-get install nginx``` and your server is up and running.
 2. cd to the nginx configuration file: ```cd /etc/nginx/sites-available/```
 3. create a reverse proxy file in the folder: ```sudo nano reverse-proxy``` and include following code:
 ```
 server {
    listen 80;
    server_name 34.252.106.230;

    location / {
        proxy_pass http://34.252.106.230:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
This configuration will listen on port 80 for requests to your domain and forward them to the local port 3000

4. Enable the configuration: Create a symbolic link from the sites-available directory to the sites-enabled directory to enable the new configuration file:
```
sudo ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/
```

5. check configuration file for errors

```
sudo nginx -t
```
6. relaod your nginx

```
sudo systemctl restart nginx

```
7. make sure you're in the app folder and type ```npm install``` and ```node app.js```.  Then you can access app from webbrowers using ip address.




## two tier architecture deployment

Want to eliminate single point of failure. This can be done using a two-tier achtitecture.

Follow the normal steps to launch your ec2 instance and ssh into it. Once done,

```sudo apt install nginx -y```

You should then be able to use public ip address in webbrowers and see the nginx brand.

To copy a folder locally to your ec2 instance enter the following command into a separate terminal:

``` scp -i "tech221.pem" -r ~/Documents/tech221_test/app ubuntu@ec2-34-252-106-230.eu-west-1.compute.amazonaws.com:/home/ubuntu/  ```

The scp command first needs to know where your pem file is located, then where the folder you want to transfer is located then finally, where on the ec2 instance you want to save the file.

Here are the steps that need to be taken:

- copy app
- install required dependencies
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

 cd into app then run the following commands:
- ```npm install``` - used to install Node.js packages or dependencies for a Node.js projec
- ```node app.js``` - is used to run a Node.js application called "app.js

Once done, you should be able to access the app by typing the following in a webbrowser: publicip:3000

#### reverse proxy with the app

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

### adding database
 - create another server db-sever
 - amend required sg rules - 27017 mongodb for your app and ssh from your ip
 - provision.sh to mongo installation set up required version
 - mongodb.conf to change the bind ip to either app ip or all
- restart then enable
- bacck to app manchie 
- create env variable called DB_HOST = db-ip
- navigate to app folder and relaunch app
- mongodb server to allow app ip

we dont want the public access to the db but give access to your app to the database

1. create a db server
Create an instance on aws using this AMI: ami-07b63aa1cfd3bc3a5. Things to note: 
- typical dont enable public ip
- the SG:

<img width="965" alt="image" src="https://user-images.githubusercontent.com/118978642/234047738-237a8631-4d09-433b-89d9-e559b47508c9.png">


2. run following commands

These commands should be run in db:
- ```sudo apt update -y```
- ```sudo apt upgrade -y```
- ```sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927```
- ```echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list```
- ```sudo apt update -y```
- ```sudo apt upgrade -y```
- ```sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20```
- ```sudo systemctl start mongod```
- ```sudo nano /etc/mongod.conf```

change bindIP:0.0.0.0

```sudo systemctl restart mongod sudo systemctl enable mongod```: if we restart vm it will put changes into effect

to check that the changes have been made do ```sudo systemctl status mongod```

go to the app instance: 
cd to home 
```sudo nano .bashrc```

insert the following into the bottom of the file: ```export DB_HOST=mongodb://<ip_address_db>:27017/posts```

```printenv DB_HOST```

```source .bashrc```

cd to app folder:
```node seeds/seed.js```
```node app.js```

go to webbrower and type ipaddressforapp/posts

#### AMIs


- ensure/posts works
- Build AMIs for nodejs server and mongod
- SG rules for app instance and db instances 
- app info for third party: port 3000 as well as port 80 and needs reverse proxy enabled in the Ireland
- db info for third party: port 27017 they wont need to ssh - they just need the private ip address to put in the env for app. NEVER NEED TO ACCESS DB
- mongod.conf changed to 0.0.0.0

Launch your db instance and check mongod is running  - if it is running then move on to check the app

Launch you app instance



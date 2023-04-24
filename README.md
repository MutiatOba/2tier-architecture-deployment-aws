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






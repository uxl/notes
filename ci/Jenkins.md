#Jenkins Configuration
On AWS EC2 Instance
http://www.bogotobogo.com/DevOps/Jenkins/Jenkins_on_EC2_1_setting_up_instance.php

#setup github
https://archive.sap.com/documents/docs/DOC-70265


Be sure to set security group to open ssh and http


You can edit your sudoers file, probably with a sudo visudo and add the line
$USER ALL=(ALL) NOPASSWD: ALL
where $USER is your own username

By the way, sudoers is in /etc/sudoers but it is dangerous to edit that file directly and possibly impossible unless you are root. It is better to use visudo since it writes to a temp file and then if everything went ok it will overwrite after it is finished writing. If vi is tricky, I suggest being VERY careful =D


https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu

```ssh ubuntu@ec2-52-203-158-164.compute-1.amazonaws.com```

#Install
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
```

```
sudo aptitude -y install nginx
```
```
cd /etc/nginx/sites-available
sudo rm default ../sites-enabled/default
```
```
sudo cat > jenkins
upstream app_server {
    server 127.0.0.1:8080 fail_timeout=0;
}
```
```
server {
    listen 80;
    listen [::]:80 default ipv6only=on;
    server_name ci.yourcompany.com;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;

        if (!-f $request_filename) {
            proxy_pass http://app_server;
            break;
        }
    }
}
```
^D # Hit CTRL + D to finish writing the file
```
sudo ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled/
```
```
sudo service nginx restart
```
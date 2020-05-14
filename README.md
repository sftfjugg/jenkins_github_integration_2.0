# Jenkins Github and Docker Containers Integration + Job chaining in Build Pipeline View of Jenkins.
![Jobs](/images/view.jpg)

For the required container images, pull them from https://hub.docker.com/
```
# docker pull a4ankur/jenkins:latest
```
And for the remaining images, you can simply pull official public images from https://hub.docker.com/

## This repo contains following tasks.
1. Create container image that’s has Jenkins installed  using Dockerfile 

2. When we launch this image, it should automatically starts Jenkins service in the container.

3. Create a job chain of job1, job2, job3 and  job4 using build pipeline plugin in Jenkins 

4. Job1 : Pull  the Github repo automatically when some developers push repo to Github.

5. Job2 : By looking at the code or program file, Jenkins should automatically start the 
	respective language interpreter install image container to deploy code 
	( eg. If code is of  PHP, then Jenkins should start the container that has PHP already installed ).

6. Job3 : Test your app if it  is working or not.

7. Job4 : if app is not working , then send email to developer with error messages.

8. Create One extra job Job5 for monitor : If container where app is running. 
	fails due to any reason then this job should automatically start the container again.
## Solutions for above tasks:-
Dockerfile to build the image.
```
FROM centos:latest
RUN  yum install java  -y --nogpgcheck
RUN yum install wget -y --nogpgcheck
RUN  wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
RUN rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
RUN yum install jenkins -y 
RUN yum install sudo -y --nogpgcheck
RUN echo "jenkins ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers
RUN yum install git -y
RUN yum install python3 -y
COPY send_mail.py /scripts/send_mail.py
CMD  java -jar /usr/lib/jenkins/jenkins.war
EXPOSE  8080
```
Python script for mailing.
```
#!/usr/bin/python3

import smtplib, ssl
#import getpass  # to take secured stdin for password

port = 587  # Port for starttls
smtp_server = "smtp.gmail.com"
sender_email = "my@gmail.com"
receiver_email = "your@gmail.com"
password = "Type your password and press enter"
message = """
Subject: Notice

This message is sent from Python.
"""
try:
    context = ssl.create_default_context()
    with smtplib.SMTP(smtp_server, port) as server:
        server.starttls(context=context)
        server.login(sender_email, password)
        server.sendmail(sender_email, receiver_email, message)
   
#except Exception as err:
#   print('Error Occured : ', err)
except:
    print("Error occurred")
else:
    print("Mail has been sent successfully")
```

After creating above image using
```
# docker build -t <tag-name> <path to Dockerfile>
```
Run the below command to run the container and access docker commands installed on host VM from the guest Jenkins-container.

-p (option for Port Address Translation)
-v (for mounting the volumes)
```
# docker run -d -v /workspace_host/:/workspace_container/ -p 8081:8080 -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker jenkins:latest
```
After this, configure your jenkins using the dashboard url of jenkins. http://<hostVM-IP>:8081 This 8081 port is binded to the 8080 exposed port of the container. To get the admin password of the jenkins, run this command,
```
# docker exec <container_ID> <path to the jenkins admin pass>
```
###### Now Create jobs for building job chaining.
## Job1
For github-webhooks's payload url, use ngrok or any other tunneling to create a tunnel which gives you a random public url binded through a publicIP behind the scene.
```
#./ngrok http 8081
```
![ngrok](/images/ngrok.jpg)

Then go to your repo->settings->Webhooks.
![Job1](/images/job1.jpg)

## Job2
Create a dtype_script.py  python script to know the file type in repo.
![dtype_script.py](/images/dtype.png)

![Job2](/images/job2.jpg)

## Job3
![Job3](/images/job3.jpg)

## Job4
![Job4](/images/job4.jpg)

## Job5
![Job5](/images/job5.jpg)

The hash in the token is generated using, 
```
# sha256 sum <write-any-thing>
```
----------------------------------------------------------------------------------------------

This repo also has some local hooks, e.g. post-commit, stored at .git/hooks/post-commit
assuming a fast-forward merge.
```
# vi  .git/hooks/post-commit

#!/bin/bash
echo "post-commit tasks are started"
git fetch
git push
echo "post commit tasks are done"

# chmod +x .git/hooks/post-commit
```

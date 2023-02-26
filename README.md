创建使用Dockerfile安装Python3和Keras或NumPy的容器映像

当我们启动镜像时，它应该会自动开始在容器中训练模型。

使用Jenkins中的build pipeline插件创建job1、job2、job3、job4和job5的作业链

Job1：当一些开发人员将repo推送到Github时，自动拉Github repo。

Job2：通过查看代码或程序文件，Jenkins应该自动启动安装了相应的机器学习工具或软件的映像容器，以部署代码并开始培训（例如，如果代码使用CNN，那么Jenkins应该启动已经安装了CNN处理所需的所有软件的容器）。

Job3：训练你的模型和预测准确性或指标。

Job4：如果度量精度低于95%，那么调整机器学习模型架构。

Job5:重新训练模型或通知正在创建最佳模型

为monitor创建一个额外的job6：如果应用程序正在运行的容器。由于任何原因失败，则此作业应自动重新启动容器，并且可以从上次训练的模型中断的位置开始。

https://blog.csdn.net/deephub/article/details/106516922

https://mp.weixin.qq.com/s/wVfLEv2YwLlRnNF9fPIYug

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
```
import os
import subprocess as sp
def f1(dir):
    for File in os.listdir(dir):
        if File.endswith(".php"):
            return "php"
        elif File.endswith(".py"):
            return "python3"
print(f1("/workspace_container/"))
```
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

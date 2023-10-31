# Introduction
A simple static website deployed using an Apache http web server within an EC2 Linux instance for demoing AWS in the MLSA Stark Expo Event. 

# Prerequisites

- Go to [AWS](https://aws.amazon.com/) and create a new account if you don't have one. This project will stay entirely within AWS Free Tier limits which is applicable to new accounts for upto 12 months since creation.
- Download the website files from this repository

# Instructions

1. After logging in, go to the Services tab in the AWS Console homepage.
2. Search EC2 and scroll to `Launch Instance`
<details>
  <summary>EC2 Instructions</summary>

  ### Creating a Linux EC2 instance
  
  1. Choose a name for your EC2 server.
  2. Under `Application and OS Images (AMI)` choose Ubuntu. This sets what OS your Linux server will be running.
  3. Under `Instance Type` choose `t2.micro`. This specifies what the technical specficiations of your server will be such as no. of CPUs, memory etc.
  4. Under `Key Pair (login)` select Create a new key pair. This gives you a key which will later be used to access your server manually and run commands and functions on it.
     * Provide a key name for example *awslinux* and leave the rest of the settings to default. Click Create key.
  5. Under `Network Settings` choose `Create security group` under Firewall. Enable allow HTTPS and HTTP traffic from the internet. This handles the security of the connections to and from the server. We have a public website so we want to allow any user(traffic) to access our site over the Internet.
  6. Under Summary, choose `Launch Instance`.
  7. Click on Instances on the left pane. The EC2 instance you just created will show up. It will take a while for it to get ready and show `Running` under the Instance State column which indicates your server is ready.
  8. Click on the Instance ID. This will open up a pane showing all the details of your server. Set aside the `Public IPv4 Address` which will later be used to access our website on the server.
  9. Click on the `Connect` button at the top of this pane. Under the Connect to Instance page, select `EC2 Instance Connect` and click `Connect`.

  ### Setting up the Website
  
  1. Once the browser based EC2 CLI opens up run the following set of commands in order.
     
     ```bash
      sudo su -
      apt-get update -y
      apt-get install -y httpd
      systemctl status httpd
      mkdir aws_site
      cd aws_site
      wget https://github.com/SourasishBasu/staticsite_starkexpo.git
      ls -lrt
      unzip main.zip
      ls -lrt
      cd staticsite_starkexpo-main
      mv * /var/www/html/
      cd /var/www/html
      ls -lrt
      systemctl status httpd
      systemctl enable httpd
      systemctl start httpd
      systemctl status httpd
     ```

     This installs an Apache HTTP server onto the Linux machine to have it behave as a web server and host the website files we transfer inside it from this git repository. The website files have been taken from [Free CSS](https://www.free-css.com/free-css-templates/page203/image-less).
</details>

3. Use the `Public IPv4 Address` from the EC2 instance to visit your website on the Internet.

## Setting up CI/CD

CI/CD is the software practice that automates software development, testing, and deployment for faster and more reliable releases.

In our case we would like to make changes to our website files within the Github repository and have those changes automatically added, deployed and reflected in the website online immediately.


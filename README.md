# Introduction
A simple static website deployed using an Apache http web server within an EC2 Linux instance for demoing AWS in the MLSA Stark Expo Event. 

# Prerequisites

- Go to [AWS](https://aws.amazon.com/) and create a new account if you don't have one. This project will stay entirely within AWS Free Tier limits which is applicable to new accounts for upto 12 months since creation.
- Download the website files from this repository.

# Instructions

1. After logging in, go to the Services tab in the AWS Console homepage.
2. Search EC2 and click on `Launch Instance`.

<details>
  <summary><h2>EC2 Instructions</h2></summary>

  ### Creating a Linux EC2 instance
  
  1. Choose a name for your EC2 server.
  2. Under `Application and OS Images (AMI)` choose Ubuntu. This sets what OS your Linux server will be running.
  3. Under `Instance Type` choose `t2.micro`. This specifies what the technical specficiations of your server will be such as no. of CPUs, memory etc.
  4. Under `Key Pair (login)` select Create a new key pair. Select the `.pem` key which can later be used to access your server manually and run commands and functions on it.
     * Provide a key name for example *awslinux* and leave the rest of the settings to default. Click Create key.
       
     ![Create key](https://i.imgur.com/x7vv9rq.png)

  6. Under `Network Settings` choose `Create security group` under Firewall. Enable allow HTTPS and HTTP traffic from the internet. This handles the security of the connections to and from the server. We have a public website so we want to allow any user(traffic) to access our site over the Internet.
  7. Under Summary, choose `Launch Instance`.
  8. Click on Instances on the left pane. The EC2 instance you just created will show up. It will take a while for it to get ready and show `Running` under the Instance State column which indicates your server is ready.
  9. Click on the Instance ID. This will open up a pane showing all the details of your server. Set aside the `Public IPv4 Address` and `Public IPv4 DNS` which will later be used to access our website on the server.
     
     ![Instance Details](https://i.imgur.com/MdJN9hV.png)
     
  10. Click on the `Connect` button at the top of this pane. Under the Connect to Instance page, select `EC2 Instance Connect` and click `Connect`.
      
      ![Connect to Instance Page](https://i.imgur.com/EaBTaHD.png)

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

In our case we would like to make changes to our website files within the Github repository and have those changes automatically added, deployed and reflected in the website online immediately using Github Actions.

Fork this repository. Inside your forked repo follow these steps:

<details>
  <summary><h2>Github Actions Steps</h2></summary>
  
  ### Creating Github Actions workflow
  1. Go to your Repository Settings > Secrets and Variables > Actions. Under Repository Secrets create the following 4 secret variables:
  
    - EC2_SSH_KEY: This will be your .pem file which you will use to login to the instance
    - HOST_DNS: Public DNS record of the instance, it will look something like this ec2-xx-xxx-xxx-xxx.us-west-2.compute.amazonaws.com from the EC2 server.
    - USERNAME: Will be the username of the EC2 instance, usually `ubuntu`
    - TARGET_DIR: Is where you want to deploy your code.
  
  2. Create a `.github/workflows` directory in your repository on GitHub if this directory does not already exist.
  3. In the `.github/workflows` directory, create a file named `github-actions-ec2.yml`.
  4. Copy the below snippet into the .yml file.

     ```bash
      name: Push-to-EC2
      
      # Trigger deployment only on push to main branch
      on:
      push:
        branches:
          - main
      
      jobs:
      deploy:
        name: Deploy to EC2 on master branch push
        runs-on: ubuntu-latest
      
      steps:
      - name: Checkout the files
        uses: actions/checkout@v2
      
      - name: Deploy to Server 1
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.EC2_SSH_KEY }}
          REMOTE_HOST: ${{ secrets.HOST_DNS }}
          REMOTE_USER: ${{ secrets.USERNAME }}
          TARGET: ${{ secrets.TARGET_DIR }}
      
      - name: Executing remote ssh commands using ssh key
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST_DNS }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo apt-get -y update
            sudo apt-get install -y apache2
            sudo systemctl start apache2
            sudo systemctl enable apache2
            cd home
            sudo mv * /var/www/html
     ```
  
  In the above block we have defined our job with name **Deploy to EC2** and enforced it to run on latest Ubuntu version. In order to deploy the code to our server and change the files on it, we need to access the EC2 instance using `ssh`. This workflow ensures that these actions only run when we push our changes to `main` branch.
</details>

Before Changes             |  After Changes
:-------------------------:|:-------------------------:
![Imgur](http://i.imgur.com/231Phxj.png)  |  ![Imgur](https://i.imgur.com/odxHUzJ.png)


## Potential Improvements

- Attach a custom domain from AWS Route 53
- Generate custom dashboards from logs generated by the instance for monitoring purposes via AWS Cloudwatch
- Create a billing alarm to alert in case of charges accumulate
- Host website on AWS S3 via Static Site Hosting


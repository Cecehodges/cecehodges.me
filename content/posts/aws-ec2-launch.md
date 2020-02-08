---
title: "Launching an AWS EC2 Instance"
date: 2020-02-07T14:37:53-06:00
draft: true
tags:
- EC2
- beginner
- SSH
- aws console
- aws 
- practice
- walkthrough
---
This post is to help you learn how to launch an EC2 instance in the AWS console. It is free to sign up for an AWS account, though you do have to enter a credit card. There is a way to set up budget that will alert you if you are getting close/are going to go over a certain amount. To learn more about budgets and how to set them up, AWS has excellent documentation [here](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/budgets-create.html). 

* Once you have logged into the AWS console, you will see a "Find Services" search bar. Type in "EC2" and select the result that pops up in the drop down feature. This will take you to the main EC2 page.

* There will be a "Launch instance" button on the main EC2 page. Once you click it, it will next ask for you to choose an Amazon Machine Image (AMI). Amazon Linux 2 is free tier eligible, so let's choose that one for now. 

* Next is to choose an Instance type. Since this is for practice and we are not requiring a ton of memory or network performance, we will also choose the free tier eligible t2.micro. Next we will configure instance details. 

* There are many options on this page for customization, but for now we will leave the default settings and click next onto Add Storage.

* Here is where we could potentially add more storage/Elastic Block Storage (EBS) volumes. For my LVM practice I did add an additional volume by clicking "Add New Volume." I left all the default options. 

* Next we can add tags. This is optional and can be skipped. If you want to add a name to your instance, click "Add Tag." Under key, enter "Name" and under value, enter the name you wish to give your instance. Click Next: Configure Security Group.

* We want to make sure that we are able to access our EC2 instance from our terminal, so we need to make sure that in our security group, SSH is available on port 22. If this is your first time in the console, the option for creating a new security group with this specification will be the default. Let's review and launch. 

* There is usually a warning at the top stating that our instance is available to the world. That's ok, because we will need our key pair to SSH into our instance. 

* When you click "Launch" a window will pop up prompting you to either create a new key pair, or that you already have a downloaded key pair. This is very important, especially if it is your first time downloading a new key pair. Once you download it, and click Launch Instances, you will **not** be able to download it again. If you accidentally delete your key pair, you will not be able to SSH into your instance. 

* It only takes a few minutes once we click Launch Instance for it to be ready for us to SSH in to. We will see a green circle stating that it is running in the status section of our EC2 console when it is ready. 

**SSH'ing into our Instance**

When you click on the instance, a "details" section will appear underneath it. We need to copy our public IPv4 address. Now open a terminal. 

We will use the IP address in the following command:
```
# ssh -i {full path of your keypair file in your computer} ec2-user@{instance public IPv4 address}
```

If you get an error saying the connection was refused, sometimes we just need to wait a minute or two and try again.

Next it will ask in your terminal the authenticity of the host, type yes and press enter. This will add that public IPv4 address to known hosts.

Now we are in our EC2 instance! If my instructions were difficult to follow, AWS has amazing documentation on all of their products, including things we have covered here. For more help on launching an instance, here is a link to the documentation that helped me [here](https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine/).




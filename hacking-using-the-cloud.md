# Hacking Using the Cloud

For more information about the cloud, please take a look at my notes on [cloud foundations]('https://github.com/puzz00/cloud-foundations/blob/main/cloud-foundations.md')

This set of notes will focus specifically on *using* the cloud to help us with our hacking activities.

## Cloud Basics

There are lots of cloud providers but since Amazon AWS is the most popular one, it lets us set up a kali linux attacking machine easily and it offers free resources, we will be using this provider in these notes.

We can have access to some AWS resources free for one year.

### Signing Up with AWS

>[!NOTE]
>We do need to provide credit card details when signing up to AWS even for using free resources

We need to visit the sign up page for AWS and provide a valid email address along with a username.

![sign up](/images/1.png)

During the process we need to choose a *support plan* - the free one is fine so we can choose it.

![sign up 2](/images/2.png)

### Installing Kali Linux with AWS

We will install an instance of kali linux on an AWS *elastic computer* aka *ec2* so we can use it as an attacking machine on the cloud.

First of all, we need to find the *Build a solution* part of the AWS home console page. We can then *Launch a virtual machine*

![ec2 3](/images/3.png)

We can then search for *kali* and when we look in the *AWS Marketplace AMIs* tab we will find the latest image for kali linux which we can then select.

![ec2 4](/images/4.png)

Once we have selected the kali linux AMI we can choose the instance type - there should be one or two which are available to the free tier so we can choose whichever one of these offers us the most resources - number of cpus and memory.

![ec2 5](/images/5.png)

We then need to create a *keypair* so we can access the instance using SSH - the *public* key will be saved to the remote kali machine whilst the *private* key will be downloaded to our local machine.

![ec2 6](/images/6.png)

The network settings can be left as they are at default values but we might be able to increase the amount of storage we have - check what the maximum amount is for the free tier and then set the amount you want to be the same as it - at the time of writing these notes that is 30GB

We can then launch the instance by clicking on the *Launch instance* button.

![ec 7](/images/7.png)

### Connecting to the Remote Machine using SSH

We need to first of all change the permissions of the *private* key which was downloaded to our local machine during the process of creating an instance of kali linux in the cloud.

```bash
chmod 700 kali_keypair.pem
```

We can now use this key along with SSH to connect to the remote kali machine - we will need to specify its public IP address which is visible in the AWS console.

```bash
sudo ssh -i kali_keypair.pem kali@11.22.33.44
```

We now have an instance of kali linux up and running from the cloud - we can access it via SSH from our local machine and we can therefore start to use it to help us with our hacking activities :smiling_imp: 

## Phishing

We can replicate any websites login page and then serve our fake page from our cloud computer. We can then socially engineer people into entering their credentials on our fake page so we can steal them - this is the essence of phishing.

### Getting Started

Before creating a phishing webpage, we need to make sure that we have a web server on our remote machine - it might not have *apache2* installed on it by default. We can check this using: `which apache2` and if no service with this name is found we can update the repositories and then install apache2 along with php like so:

```bash
sudo apt-get update
sudo apt install apache2 php
```

We will then need to start the apache2 web server using: `sudo service apache2 start`

Next, we will need to add a new rule to our remote machines security group to allow inbound connections on port 80.

![phishing1](/images/8.png)

![phishing2](/images/9.png)

Once we have added this new rule our web root will be on the internet - we can use this to host our malicious webpages and apps.

### Cloning Websites

We can clone a webpage by simply navigating to it and then right clicking on it before selecting `Save Page As` We can then create a local copy of the source code of the webpage we want to use as a base for our phishing page. In this example, we are going to use *Astrobin*

If this does not work - sometimes the webpage renders in a weird way - we can use `httrack` to download the resources we need.

The easiest way to use this tool is to just run it and then after specifying an output directory for the downloaded resources we can continue by selecting to mirror the website using the wizard - this is currently option 2 - the default options can be left the same.

>[!NOTE]
>This tool will take a variable amount of time to mirror the website - it can be very quick but for larger sites it can take a while

#### Using Filezilla for File Transfers

We can use Filezilla to interact with our remote kali machine via a GUI which makes it easier to transfer and work with files. We can use SSH with Filezilla so our data is still secure.

We can use our package manager to install Filezilla:

```bash
sudo apt-get update
sudo apt install filezilla
```

We can then connect to the remote kali machine via Filezilla.

![phishing2](/images/10.png)

>[!IMPORTANT]
>We need to use `sudo chown kali:kali /var/www/html -R` via SSH to change the ownership of the web root on the remote kali machine before we can transfer files to it

We can now transfer the mirrored website files to our remote kali machine by simply dragging and dropping them into `/var/www/html` from within Filezilla.

>[!NOTE]
>Make sure `index.html` is copied across to the web root and the default apache2 `index.html` page has been deleted

![phishing3](/images/11.png)

Once we have completed all of the above steps we can test if it has worked by navigating to the IP address of our kali machine in the cloud - we should see a replica of the login page we are using for our phishing endeavours.

![phishing4](/images/12.png)


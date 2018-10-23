# Deploying a Frontend React Project on Amazon Lightsail

*A guided tutorial on how to host a frontend React project using Amazon Lightsail*

## What is Amazon Lightsail?

Amazon Lightsail is part of Amazon Web Services, which is itself a subsidiary of Amazon.com (gotta love corporate infrastructure). Lightsail allows you to rent cloud servers, or 'virtual machines', with many attractive low-cost choices. It has easy options for scaling your hosted projects as more and more people use them, and integrates with many of the other great services AWS provides. Setting up a Lightsail server is relatively simple and can be accomplished quickly, assuming you don't run into serious issues.

This article will focus on all the steps necessary to host a frontend-only (no server/database) React project on Amazon Lightsail. If your project isn't purely frontend, this guide will still help you most of the way. Extra steps will be required, such as daemonizing your server and set up nginx as a reverse proxy.

## Step 1: Purchase your virtual machine

[Head here to see all the available options.](https://aws.amazon.com/s/lp/epid1014-b/?trk=ps_a131L000005Of2gQAC&trkCampaign=ACQ_Amazon_Lightsail&sc_channel=ps&sc_campaign=acquisition_US&sc_publisher=google&sc_category=lightsail&sc_country=US&sc_geo=NAMER&sc_outcome=acquisition&sc_medium=ACQ-P|PS-GO|Brand|Desktop|SU|Compute|Lightsail|US|EN|Text|Lightsail&s_kwcid=AL!4422!3!301788508049!e!!g!!amazon%20lightsail&ef_id=Wy72MwAAAgfjX-ds:20181018175212:s) 

We will be working with Linux servers. Pick one that you can afford or that you think will fulfill your needs for now, and keep an eye out for any specials or promotions (free first month, etc). You can always upgrade your virtual machine's capabilities later on if you need to.

Next, create an AWS account if you don't already have one and enter payment information to gain access to your Lightsail home page. Once there, click the 'Create an Instance' button to set up the virtual machine. You'll want to select the 'Linux/Unix' option, as well as the 'OS Only' toggle. The Linux flavor this guide will cover is 'Ubuntu', so make sure that one is highlighted. Double check that the correct price and configuration is selected, then enter a custom name for this instance if you wish. Finally, click 'Create Instance' to finalize the process.

## Step 2: Configuring SSH Login

### Setting a Static IP

You should now see your machine's status card in the 'Instances' tab of Lightsail Home. Before moving on, we need to set a static IP address for our instance so we don't have to re-enter a new IP any time it changes. Navigate to the 'Networking' tab and click 'Create static IP'. Follow the instructions to link the IP address to your newly created instance. Then, head back home and click on the terminal button on the status card (it has a '>_' on it). Put on your hacker hat and sunglasses because you're in!

### Adding New Super User

Now that you're logged in, let's add a custom user to the system. This profile will be the one you'll use to login and manage your machine in the future. The examples given will apply 'username' as a placeholder for your desired name. Enter the following command:

```
sudo adduser username
```

Set a strong UNIX password as prompted, and enter user information as desired (it's fine to leave these fields blank).

You just created a basic user on your virtual machine. Let's grant it 'superuser' capabilities to make our lives easier going forward. This will enable the `sudo` command on the profile so you can have administrative privileges:

```
usermod -aG sudo username
```

### Create and Copy Public Key

Next, we shall secure our server by requiring an SSH key to log in to the new user. If you don't already have an SSH key pair, open a new terminal on your local computer and enter:

```
ssh-keygen
```

Something like the following will output to the terminal:

```
ssh-keygen output
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/localuser/.ssh/id_rsa):
Hit return to accept this file name and path (or enter a new name).
```

The system will ask you for a passphrase to secure they key. You can chose to enter one, or simply leave it blank. 

Now you have your own private/public key pair in the directory that was specified above. Use this command to print your public key (id_rsa.pub) to the terminal:

```
cat ~/.ssh/id_rsa.pub
```

You'll see something like this: 

```
id_rsa.pub contents
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBGTO0tsVejssuaYR5R3Y/i73SppJAhme1dH7W2c47d4gOqB4izP0+fRLfvbz/tnXFz4iOP/H6eCV05hqUhF+KYRxt9Y8tVMrpDZR2l75o6+xSbUOMu6xN+uVF0T9XzKcxmzTmnV7Na5up3QM3DoSRYX/EP3utr2+zAqpJIfKPLdA74w7g56oYWI9blpnpzxkEd3edVJOivUkpZ4JoenWManvIaSdMTJXMy3MtlQhva+j9CgguyVbUkdzK9KKEuah+pFZvaugtebsU+bllPTB0nlXGIJk98Ie9ZtxuY3nCKneB+KjKiXrAvXUPCI9mWkYS/1rggpFmu3HbXBnWSUdf localuser@machine.local
```

Select and copy the public key, beginning with the `ssh-rsa` and ending with the email.

### Add Key to Lightsail Machine

Back on the Amazon Lightsail terminal, switch from the root 'ubuntu' user to your new one (substitute your own username):

```
su username
```

You're now in your user's home directory.

Make a new directory called .ssh and restrict its permissions with these commands:

```
sudo mkdir ~/.ssh
sudo chmod 700 ~/.ssh
```

Next, use the 'nano' text editor to create and open a new file in .ssh:

```
sudo nano ~/.ssh/authorized_keys
```

Insert your public key here by first pasting the key into the clipboard (click the icon on the bottom right of the window), and right clicking in the empty file. To exit hit CTRL-x, then y to save, and ENTER to confirm the file name.

Restrict the permissions of the authorized_keys file for additional security:

```
sudo chmod 600 ~/.ssh/authorized_keys
```

### Log in Using Your Computer

You now have the ability to log in to your server from your local machine. On your computer's terminal, enter this (substituting your server's username and IP address):

```
ssh username@your_server_ip
```

After submitting your password, you should be logged in to the same server as your Lightsail terminal. Go ahead and close the Lightsail terminal - you've got remote access now, you cheeky computer wizard!

### Configure Firewall

AllowSSH connections through the server's firewall by typing:

```
sudo ufw allow OpenSSH
```

Then activate the firewall with:

```
sudo ufw enable
```

## Step 3: Point Domain Name to Lightsail Server

Find a domain name provider to purchase your desired domain if you haven't already. Afterwards, navigate to the DNS settings and add/update the A (@) record's value to be your server's IP address. If there are any AAA record types, remove them now to save yourself from a headache later.

Depending on the provider, this setting can take up to 48 hours to be completely applied. However, it usually takes less than 15 minutes.

## Step 4: Install NGINX

NGINX is a super powerful and popular web server that you can use to serve your website's files. Over 400 million websites use NGINX open source - know that your project is going to be using one of the best services out there.

Back on your terminal, update your package listings and install the latest version of NGINX with:

```
sudo apt-get update
sudo apt-get install nginx
```

Allow NGINX access through the firewall by typing:

```
sudo ufw allow 'Nginx Full'
```

## Step 5: Install Node.js and NPM

Use Ubuntu's 'curl' tool to download the Node setup script:

```
sudo curl -sL https://deb.nodesource.com/setup_10.x -o nodesource_setup.sh
```

Then, run the script with sudo:

```
sudo bash nodesource_setup.sh
```

After setup, install the latest Node.js package with:

```
sudo apt-get install nodejs
```

Npm is installed alongside Node.js here. Lastly, install the build-essential package to ensure that all of our npm packages will work correctly:

```
sudo apt-get install build-essential
```

## Step 6: Add Project Files to Lightsail Server

Navigate to the home directory and clone your files from GitHub using these commands (make sure to swap out your own username and repo link):

```
cd ~
git clone https://github.com/your-username/your-repository-name.git
```

Now add all your dependencies by going into the project directory and using npm install. **If you have a weaksauce server (512 MB RAM or less), follow the extra steps (6.5) below before running `npm i`!** Otherwise, pay no attention to those and simply run:

```
cd your-repository-name
npm i
```

Afterwards, run the build script:

```
npm run build
```

## Step 6.5: Extras - Making a Swap File

If you are reading this, you probably chose the most frugal option Amazon Lightsail has to offer. There's nothing wrong with that - everyone loves to save a bit of money. The bad news is that your server doesn't have enough RAM to `npm install`. The good news is that there's a super cool workaround you can do for this in Ubuntu.

To solve our issue, we will be making a 'swap file' that offloads data. This special file allows an Ubuntu machine's hard drive to take over when the RAM is full, which is perfect for our situation. Using a hard drive to transfer data takes a lot longer than relying on RAM alone, but there's really no other option here.

First, navigate to the root directory and create a 1 gigabyte swap file using the fallocate program:

```
cd /
sudo fallocate -l 1G /swapfile
```

For security, restrict this file's permissions to `root`:

```
sudo chmod 600 /swapfile
```

Then, mark this file as swap space:

```
sudo mkswap /swapfile
```

You should see something like this in the output: 

```
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=6e965805-2ab9-450f-aed6-577e74089dbf
```

Finally, enable the swap file for immediate use:

```
sudo swapon /swapfile
```

If you enter this command, you should see the swap file ready to go:

```
sudo swapon --show
```

If you have an output from the previous command, you can now go back to your project folder and `npm i`:

```
cd home/username/your-repository-name
npm i
```

After installation finishes (this may take a while), make your build folder with:

```
npm run build
```

Now we need to clear our swap file configs. Amazon Lightsail uses solid state drives for its hard storage, which is great for quick performance. However, SSD's are not built to be repeatedly overwritten like traditional spinning disks. For this reason, swap files are a hazard to SSD's and may quickly degrade the performance of your virtual machine's hardware.

To remove the temporary swap file, restart your server from the Lightsail home page by opening the menu on your instance and clicking 'Reboot'. Wait a minute or two, then log back in on your local machine's terminal using:

```
ssh username@your_server_ip
```

## Step 7: Link the Project's Build Folder in the Server

NGINX will be looking for your website's static files in a certain directory. The default location will be in `/var/www`. Navigate there and create a new folder to hold your project's build files (again, use your own values for placeholder entries).

```
cd /var/www/
sudo mkdir your_domain.com
```

Now, make a symbolic link to your project's build folder with the `ln` command:

```
sudo ln -s /home/username/your-repository-name/build/* /var/www/your_domain.com/
```

This will mirror all the information from `your-repository-name/build` to the `your_domain.com` folder. It's sort of like a live copy/paste that will update itself if the original file changes.

## Step 8: Modify the NGINX Configuration Files

NGINX should be installed and ready to go, but we need to configure it to serve our website's files. There are two directories in NGINX that we will modify: `sites-enabled` and `sites-available`. Again, we will use nano to create and edit a new file. Don't forget to replace the fake domain names with your own!

```
cd /etc/nginx/sites-available
sudo nano your_domain.com
```

Delete the default text in this file (you can do this quickly by placing your cursor at the beginning and holding CTRL-k). Then, enter the following information:

```
server {
 listen 80 default_server;
 listen [::]:80 default_server;
 root /var/www/your_domain.com;
 index index.html;
 server_name your_domain.com www.your_domain.com;
 location / {
   try_files $uri $uri/ /index.html;
 }
}
```
This server block tells NGINX to listen on port 80, as well as specifying where the files to serve are located with `root` and `index`. `server_name` accepts your domain names (one with `www` and one without) to route requests to these files. The `location /` entry makes sure that any paths not available in the project route the user back to `index.html`.

Exit the nano editor same as before - CTRL-x, then y, then ENTER.

Instead of tediously backing out and retyping the information in `sites-enabled`, let's make another symbolic link:

```
sudo ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/your_domain.com
```

Check your work for syntax errors by running:

```
sudo nginx -t
```

If the test is successful, restart your server:

```
sudo systemctl restart nginx
```

Given that your domain provider has fully propagated the changes you made earlier, you should be able to view your site by entering your domain name in a browser! So awesome!

## Step 9: Obtain SSL Certificate with Let's Encrypt

Getting an SSL certificate to enable HTTPS used to be a complicated and potentially expensive process. Luckily, an organization called Let's Encrypt came along to provide free TLS/SSL certificates and helpful tools to activate them. We will be using one of their programs called CertBot.

Add the CertBot repository for the latest version:

```
sudo add-apt-repository ppa:certbot/certbot
```

Update the package list afterwards to pick up the new information:

```
sudo apt-get update
```

Lastly, install CertBot's python/NGINX package:

```
sudo apt-get install python-certbot-nginx
```

This plugin will do most of the work needed to get and maintain a security certificate, including modifying the NGINX configuration files and reloading when they eventually expire. To run CertBot, enter:

```
sudo certbot --nginx -d your_domain.com -d www.your_domain.com
```

Enter your email address and agree to the terms of service if required. CertBot will then present you with some options:

```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```

You'll want to type `2` and press ENTER. If everything went smoothly, CertBot will update your configs, reload NGINX, and confirm success with a message.

## Step 10: Allow HTTPS Port on Lightsail Instance

Obtaining an SSL certificate with CertBot reconfigures your server to run on secure port 443 instead of the default 80. We need to update our Lightsail records to allow this. From the home page, click on the menu of your instance and go to 'Manage'. Then, click on the 'Networking' tab. Add a new value to the 'Firewall' section. If you select 'HTTPS' from the 'Value' dropdown, it should automatically make the 'Port Range' 443. Save your changes, and you're all set!

## Conclusion

The ability to host your own projects is an important skill as a web developer. Amazon Lightsail offers you a great and inexpensive way to get started. By following this guide, your website should be up and running in no time!
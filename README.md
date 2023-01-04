# Valheim In the Cloud

Technologies used: AWS | Linux | EC2 | steamCMD | 

For this project I'll be walking you through the process to create a dedicated server for the steam game Valheim - all entirely hosted in the Amazon Cloud. While you can host a server on your own computer as you play, hosting it in the cloud will give you better performance (FPS) while also allowing any friends to play on your server without you being online. 

EC2 instances will be charged at an hourly rate as per https://aws.amazon.com/ec2/pricing/on-demand/ or at a discount if you use dedicated pricing. For an optimised experience with multiple players, I recommend using the a1.xlarge instance type. 

For the purposes of this guide we will be using Free Tier resources which will not result in charges to your account. This guide will also assume minimal knowledge about the cloud so explanations may be verbose.

Part 1: Setting up your AWS EC2 Linux Instance

Part 2: Setting up your Valheim Server

Part 3: Cost Optimizations: Auto-scaling your resources when not playing (work in progress)

## Part 1: Setting up your AWS EC2 Linux Instance

Begin by making an account at  https://aws.amazon.com/ 
Once your account is verified, the first thing you should do is enable MFA (multi factor authentication) as a security practice. 

On the aws console https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1 use the search bar in the top left. Enter "IAM" (Identity access management). Once you are on the IAM dashboard click "Add MFA" near the top right of the page. 

Click on "Activate MFA" in the "Multi-factor authentication (MFA)" tab and create a name, select "virtual MFA device", and continue. If you are using android you can use google's official authenticator app as your phone's 2FA device, or click on the list of compatible applications to choose your preferred app. Complete setup via instructions and return to the aws console.

Next, we will be launching our EC2 instance which will function as our Valheim server. Navigate to the top left search bar and enter "EC2" 

On the left sidebar click "Instances" under the instances tab. This is where you are able to monitor your EC2 instances. It provides plenty of useful information which we will need later, but for now we will launch our server instance.

Click "Launch instances" in the top right. Give your instance a name; for this blog we will name it **Valheim-Server**. Select Ubuntu 22.04 for your AMI.
Next is the instance type. For the purposes of this blog we will be using the t2.micro as it is free to use. However, for optimal performance an a1.xlarge instance would fit; review the pricing accordingly. 

Next, create a key pair. **This is very important**. This is a file that acts as the password to your instance so make sure you create it and keep it secure. For the purposes of this blog the keypair will be named **Chinese Electric Batman Keypair**. 

Next, under network settings, we will create a security group; this acts as a firewall for our instance. Click "Edit" to open more advanced options. Give your security group a name; ours will be **Valheim-Server Security Group**. Give it a description. Under the security group rules configure the following:
For Security group rule 1: Type = ssh, Source type = anywhere, TCP, and port 22 (already configured).
For Security group rule 2: Type = custom UDP, Source = 0.0.0.0/0, port range of 2456-2458

The first security group allows you to connect to your instance, while the second opens up the UDP ports that Valheim uses to connect to players.

All other settings can be left at default. https://i.imgur.com/2BMORg6.png 

Launch the instance and then return to the instances tab. You should see your Valheim server running. You can click on the checkbox and examine details such as its public IP address. 

To connect to your instance you can either use a GUI SSH tool like PuTTY or connect using the AWS console. To connect using the AWS console select your server and connect at the top right. Choose "EC2 Instance Connect". The default username for the linux server, ubuntu, should already be filled in. Click connect and you will be in!

The last task for Part 1 is to update linux. Copy the command or type it, without quotes, "sudo apt update" and paste it into the console by right-clicking and selecting paste (or just right clicking using PuTTY). Then, "sudo apt upgrade". Read the output and accept the update.

You should get a final output and then you're finished with Part 1! Next, we'll be installing Valheim and configuring it to run. 


# Part 2: Setting up your Valheim Server

We have created and set up our EC2 linux server and implemented a few basic security measures. Next we'll be installing and configuring the Valheim dedicated server from steam. 

First, we'll create a separate user that we'll run things from (using root user is a bad idea from a security perspective).

Enter "sudo useradd -m steam" to create a new user named steam. 
Then, "sudo passwd steam", and enter your new password. 
Log into our new steam user with "sudo -u steam -s"
Then, enter our new steam directory with "cd /home/steam". Confirm our current directory using the command "pwd" - we should get the output "/home/steam".


Next, we will be installing steamCMD, a tool used to install and update dedicated servers. For more information feel free to read the wiki https://developer.valvesoftware.com/wiki/SteamCMD

Enter these commands one at a time to install the required dependencies, followed by steamCMD:

- sudo add-apt-repository multiverse
- sudo apt install software-properties-common
- sudo dpkg --add-architecture i386
- sudo apt update
- sudo apt install lib32gcc-s1 steamcmd"
- cd ~

Next, run this long command: "steamcmd +force_install_dir /home/steam/.steam/steamapps/common/valheim +login anonymous +app_update 896660 validate +exit". 

This launches steamcmd, logins in using an anonymous user, downloads Valheim (896660 is the identifier used for Valheim in steam's database), and then exits the steamCMD utility.  

Navigate to outside 

# Part 3: Cost Optimizations: Auto-scaling your resources when not playing **(Work in Progress)**
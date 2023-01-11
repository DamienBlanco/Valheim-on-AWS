# Valheim In the Cloud

Technologies used: AWS | EC2 | Linux | Shell Scripting | steamCMD | systemD

For this project I'll be walking you through the process to create a dedicated server for the steam game Valheim - all entirely hosted in the Amazon Cloud. While you can host a server on your own computer as you play, hosting it in the cloud will give you better performance (FPS) while also allowing any friends to play on your server without you being online. 

This guide will show the manual way to install and configure all resources. Faster automated methods using docker containers exist; however, this guide is intended as a fun way to learn linux and AWS features and comes with many explanations.

EC2 instances will be charged at an hourly rate as per https://aws.amazon.com/ec2/pricing/on-demand/ or at a discount if you use dedicated pricing. For an optimised experience with multiple players, I recommend using the t3a.large instance type as the latest Mistlands update has pushed the RAM usage above 6GB.


## Part 1: Setting up your AWS EC2 Linux Instance

Begin by making an account at  https://aws.amazon.com/ 
Once your account is verified, the first thing you should do is enable MFA (multi factor authentication) as a security practice. 

On the aws console https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1 use the search bar in the top left. Enter "IAM" (Identity access management). Once you are on the IAM dashboard click "Add MFA" near the top right of the page. 

Click on "Activate MFA" in the "Multi-factor authentication (MFA)" tab and create a name, select "virtual MFA device", and continue. If you are using android you can use google's official authenticator app as your phone's 2FA device, or click on the list of compatible applications to choose your preferred app. Complete setup via instructions and return to the aws console.

Next, we will be launching our EC2 instance which will function as our Valheim server. Navigate to the top left search bar and enter "EC2" 

On the left sidebar click "Instances" under the instances tab. This is where you are able to monitor your EC2 instances. It provides plenty of useful information which we will need later, but for now we will launch our server instance.

Click "Launch instances" in the top right. Give your instance a name; for this blog we will name it **Valheim-Server**. Select Ubuntu 22.04 for your AMI.
Next choose the instance type; for optimal performance I recommend a t3a.large instance; review the pricing accordingly. 

Next, create a key pair. **This is very important**. This is a file that acts as the password to your instance so make sure you create it and keep it secure. For the purposes of this blog the keypair will be named **Chinese Electric Batman Keypair**. 

Next, under network settings, we will create a security group; this acts as a firewall for our instance. Click "Edit" to open more advanced options. Give your security group a name; ours will be **Valheim-Server Security Group**. Give it a description. Under the security group rules configure the following:

For Security group rule 1: Type = ssh, Source type = anywhere, TCP, and port 22 (already configured).

For Security group rule 2: Type = custom UDP, Source = 0.0.0.0/0, port range of 2456-2458

The first security group allows you to connect to your instance, while the second opens up the UDP ports that Valheim uses to connect to players.

All other settings can be left at default. https://i.imgur.com/2BMORg6.png 

Launch the instance and then return to the instances tab. You should see your Valheim server running. You can click on the checkbox and examine details such as its public IP address. 

To connect to your instance you can either use a GUI SSH tool like PuTTY or connect using the AWS console. To connect using the AWS console select your server and connect at the top right. Choose "EC2 Instance Connect". The default username for the linux server, ubuntu, should already be filled in. Click connect and you will be in!

The last task for Part 1 is to update linux. Enter the following commands into the CLI (you can paste them by right clicking and selecting paste, or just right clicking if you're using puTTY

- sudo apt update
- sudo apt upgrade

Read the output and accept the update. You should get a final output and then you're finished with Part 1! Next, we'll be installing Valheim and configuring it to run. 


## Part 2: Setting up your Valheim Server

We have created and set up our EC2 linux server and implemented a few basic security measures. Next we'll be installing and configuring the Valheim dedicated server from steam. 

We will be installing steamCMD, a tool used to install and update dedicated servers. For more information feel free to read the wiki https://developer.valvesoftware.com/wiki/SteamCMD

Enter these commands one at a time to install the required dependencies, followed by steamCMD:

- sudo add-apt-repository multiverse
- sudo apt install software-properties-common
- sudo dpkg --add-architecture i386
- sudo apt update
- sudo apt install lib32gcc-s1 steamcmd"
- cd ~
- steamcmd +force_install_dir /home/Ubuntu/.steam/steamapps/common/valheim +login anonymous +app_update 896660 validate +exit

The last command launches steamcmd, sets the install directory, logs in using an anonymous user, downloads Valheim (896660 is the identifier used for Valheim in steam's database), verifies installation, and then exits the steamCMD utility.  

Next we'll create a simple shell script that can be run to easily update Valheim. You can navigate to inside the Valheim install directory or simply place it in the /home/ubuntu location. 

- touch valheim_update.sh
- echo 'steamcmd +force_install_dir /home/ubuntu/.steam/steamapps/common/valheim +login anonymous +app_update 896660 validate +exit' >> valheim_update.sh
- chmod -R 755 .
- You can test the script by running the following command: "./valheim_update.sh"

Next we'll be modifying the script used to start the server. 
- cd /home/ubuntu/.steam/steamapps/common/valheim - is the default used in this guide, otherwise navigate to where you installed valheim using the ls and cd commands (hidden directories need the ls -a command). 
- cp start_server.sh valheim_start.sh - This makes a copy of the shell script
- vim valheim_shart.sh - This opens the vim editor.

We will need to make some changes using the vim editor. Here is a quick guide for the commands we can use <https://coderwall.com/p/adv71w/basic-vim-commands-for-getting-started> 

Press escape to ensure you are in command mode, then press i to enter insert mode. You can navigate with arrow keys and input or delete text like normal. Edit the text by adding the extra export at the top, and make sure to change the server name and password as appropriate. **At the time of writing "--crossplay" must be deleted from the script as it breaks the server**. When you are finished, hit escape to return to the command mode, and enter ":wq!" to save and quit. 
```
export TERM=xterm
export templdpath=$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=./linux64:$LD_LIBRARY_PATH
export SteamAppId=892970

echo "Starting server PRESS CTRL-C to exit"

./valheim_server.x86_64 -name "MyServerName" -port 2456 -world "Dedicated" -password "MyPassword"

export LD_LIBRARY_PATH=$templdpath
```
We're almost done. The server is playable and connectable at this point, however it will shut down if you close the ssh connection. We will need to complete one more step so that it can run without us being logged in. This will set up a service using systemd.

- cd /etc/systemd/system
- sudo touch valheim.service
- sudo vim valheim.service

Modify valheim.service so that it looks like the following using the vim editor as before.
```
[Unit]
Description=Valheim service
Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=ubuntu
WorkingDirectory=/home/ubuntu/.steam/steamapps/common/valheim/
ExecStart=/bin/sh /home/ubuntu/.steam/steamapps/common/valheim/valheim_start.sh

[Install]
WantedBy=multi-user.target
```
If you've changed the installation directory for valheim, created a separate user to install the server on, or named your startup script something else, be sure to modify this template as appropriate.

- cd /home/ubuntu
- echo 'sudo systemctl start valheim' >> boot_server.sh
- sudo chmod -R 775 .

This will create a startup script for the valheim service. You can start it by executing the script (./ followed by script name). Alternatively, you can set the service to automatically start every time linux launches by using the command "sudo systemctl enable valheim" (disable this feature by replacing enable with disable). 

You can now run the boot_server.sh to start your server (remember, using ./ followed by the script you are executing)! Players can connect using your ec2 instance's public ip address which you can find by entering the command 'hostname -i' or via the aws instance tab. Happy playing!



## Part 3: Cost Optimizations: Auto-scaling your resources when not playing **(Work in Progress)**

# Minecraft Server Tutorial Deployment on AWS EC2

## Launch an Instance
1. **Console**
    - Login &rarr; select your desired region in the top-right.
    - Search for EC2 in the search bar and open the dashboard.

2. **Launch**
    - Click on the yellow **"Launch instance:** button
    - In the Name and Tags panel:
        -  Give the instance a name (ex. **minecraft-server**).
    - In the Application and OS Images panel:
        -  Select **Amazon Linux 2023 AMI** for the OS image
        - Select **64-bit (Arm)** for the architecture type
    - Select **t4g.small** for the instance type to ensure enough memory (2 vCPU, 2GiB RAM)
    - Create a new key pair or select a previosuly used key pair that you have access to.
3. **Configure Security Group** in the Network Settings:
    - Select a **default** VPC
    - Select **"No preference"** for the subnet
    - Select **"Enable"** for the Auto-assign public IP question
    - For Firewall Security:
        - Select **"Create a security group"**
        - Name your security group (ex. **minecraft-sg**)
        - For Inbound Rules create two seperate ones:
            1. An SSH type, TCP protocol, port range 22, "My IP" source, and label it's description as "SSH access"
            2. A Custom TCP type, TCP protocol, 25564 port range, "Anywhere" source type, and label it's description as "Minecraft client access"
4. Leave storage at **8 GiB** gp2 and leave User Data blank
5. **Review then click "Launch instance"** in the bottom right corner.
    - After launching, click on "Instances" in the top navigation bar and ensure the "instance state" is running.

## SSH into Your Instance

6. Click the **Instance ID** of your newly made instance and **copy the Public IPv4 address** of your instance for later use.
7. Open your terminal and connect via ssh:  
`ssh -i <path_to_your_key> ec2-user@<your_public_ipv4>`  
For example, I would run:  
`ssh -i /Downloads/key_pair.pem ec2-user@35.93.36.201`

## Install Java (Corretto 21)and Prepare the Server Directory
8. Run the following commands:  
`cd /tmp`  
`wget https://corretto.aws/downloads/latest/amazon-corretto-21-aarch64-linux-jdk.tar.gz`  
`sudo mkdir -p usr/lib/jvm`
`sudo tar -xzf amazon-corretto-21-aarch64-linux-jdk.tar.gz`  
`sudo alternatives --install/usr/bin/java java /usr/lib/jvm/amazon-corretto-21.0*/bin/java 2000`  
`sudo alternatives --config java`
9. **Select Corretto 21** when prompted then verify:  
`java --version`  
You should see OpenJDK 21

10. Run in your terminal:  
`mkdir -p ~/minecraft-server`  
`cd ~/minecraft-server`
11. **Get the latest server.jar** from [Minecraft Server Download](https://www.minecraft.net/en-us/download/server) and **copy the link address**.
12. Run the command:  
`wget https://launcher.mojang.com/v1/objects/e6ec2f64e6080b9b5d9b471b291c33cc7f509733/server.jar -0 server.jar`
13. Agree to the EULA by running in your terminal:  
`echo "eula=true" > eula.txt`


## Let's Run the Server
14. **Run the server** using:  
`java -Xmx2G -Xms1G -jar server.jar nogui`  
Wait for the **"Done (xx.xx s)!"** line to appear, after you see this you can stop it using **Ctrl+Z** and you will now have server.properties generated in your directory.

## Let's Setup Auto-Start
15. Ensure you are **in the minecraft-server directory**, then run:  
```vim minecraft-start.sh``` 
16. Select 'i' on your keyboard to **insert and copy** in the following script:
```
#!/usr/bin/env bash  
cd ~/minecraft-server  
java -Xmx1G -Xms512M -jar server.jar nogui
```  
Once pasted in hit the escape button on your keyboard and then type ":wq" to exit vim and save.  
17. **Change the permissions** of this file by making it executable by running in the minecraft-server directory:  
`chmod 700 start-minecraft.sh`  
18. **Test it** by running:  
`~/minecraft-server/minecraft-start.sh`  
You should see the: **"Done (xx.xx s)!"** line near the end.  

## Now Let's Configure Automatic Startup  
19. **Run** the following command:  
`sudo vim /etc/systemd/system/minecraft.service`  
20. Hit 'i' for insert on your keyboard and **paste in the following unit commands**:  
```
[Unit]
Description=Minecraft Server
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/minecraft-server
ExecStart=/home/ec2-user/minecraft-server/start-minecraft.sh
Restart=on-failure
SuccessExitStatus=0 1
TimeoutStopSec=20

[Install]
WantedBy=multi-user.target
```  
Press escape to leave insert mode and then type ":wq" to exit vim and save.  

## Reload, Enable, Start, and Check Status:  
21. **Run** the following commands:  
`sudo systemctl daemon-reload`  
`sudo systemctl enable minecraft.service`  
`sudo systemctl start minecraft.service`  
`sudo systemctl status minecraft.service`
22. You should see **Active: active (running)**, once you've seen this, your Minecraft Server will start each time your instace is spun! :D
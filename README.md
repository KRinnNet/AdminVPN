![1_adhoc.jpg](https://www.softether.org/@api/deki/files/612/=1_adhoc.jpg?size=webview)

# Set up low cost AdHoc VPN - Control Multiple Linux or Windows Servers over broadband without the need to purchase a Static IP, using SoftEther VPN!

### Introduction
Hi! I'm Darshan, in this tutorial I will explain how to make your own AdHoc VPN (where you will have one central VPN Server and multiple Clients [ Servers or Desktops or Mobile ] communicate with each one over broadband without the need to purchase or rent a Static IP from ISP ).

### Prerequisites:
To complete this tutorial, you will need access to an Ubuntu 18.04 server to host your VPN Server. You will need to configure a non-**root** user with `sudo` privileges before you start this guide. You can follow this guide [Ubuntu 18.04 initial server setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04) to set up a user with appropriate permissions. 

Basic Knowledge of Linux, Windows, Android and iOS systems (depends on which system will be clients) will be useful.

# Step 1:  Create VPN Server

To start off, spin your **VM** ( I have chosen DigitalOcean for this since they provide VM's at a cheaper price and also option for upgrade based on usage ) 

Follow this tutorial for VPN Server Installation: **How to Setup a Multi-Protocol VPN Server Using SoftEther** [https://www.digitalocean.com/community/tutorials/how-to-setup-a-multi-protocol-vpn-server-using-softether](https://www.digitalocean.com/community/tutorials/how-to-setup-a-multi-protocol-vpn-server-using-softether)
 
 Once the Server is complied and installed it is easier to maintain using the server manager available to download in SoftEther website - [https://www.softether-download.com/en.aspx?product=softether](https://www.softether-download.com/en.aspx?product=softether)

Download the appropriate manager for your OS and Install them.

# Step 2:  Configure VPN Server

You can create a 'A' record in your own domain name and point it to the VM or use the IP address of the server to login through the Server Manager.

Follow this tutorial on how to set up an AdHoc SofthEther VPN Server:

[https://www.softether.org/4-docs/2-howto/1.VPN_for_On-premise/1.Ad-hoc_VPN](https://www.softether.org/4-docs/2-howto/1.VPN_for_On-premise/1.Ad-hoc_VPN)

Also :

- Enable "L2TP over IPsec" and set and IPsec password, ( Needed if you want to connect to VPN through Android or iOS devices )
- Create a new Hub by providing custom name, for this tutorial it will be "Virtual Hub"

- Go to Manage Virtual Hub and create users to login. You can select Password Authentication for simpler use.

- **Enable Virtual NAT and Virtual DHCP** if you want the sever to act as DHCP server, if you have a DHCP server in the cluster you can skip this step.
 > **Note:** If you don't want the Server or Client from accessing the internet through VPN then remove the Default Gateway and DNS Server Address in Secure NAT Configuration.
 >
 >  If want to assign Static IP to your Server then set the DHCP start and end IP address outside of your server IP address pool.

# Step 3:  Install VPN Client

Choose the appropriate option below based on Client OS

## For Windows :
Download and install the client available in SoftEther Downloads, it's a straight-forward installation without any additional settings  :  [SoftEther Download](https://www.softether-download.com/en.aspx?product=softether)

You can follow the VPN Client Setup Manual here [SoftEther VPN Client Manual](https://www.softether.org/4-docs/1-manual/4._SoftEther_VPN_Client_Manual)

## For Linux:
Installation in Linux is a bit tricky, but much similar to VPN Server installation,

-Download the **VPN Client** Software from SoftEther here [SoftEther Download](https://www.softether-download.com/en.aspx?product=softether) , by selecting the Platform as Linux, CPU based on Client CPU Architecture and copy the download file link and paste in wget.

    cd ~
   
Download the Client Files
    
    wget [Paste the Download Link]

Extract the downloaded file

    tar xzvf softether-vpnclient-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz

After extracting it, a directory named  **vpnclient**  will be created in the working folder. In order to compile SoftEther, the following tools and packages must be installed on your server: **make**,  **gccbinutils (gcc)**,  **libc (glibc)**,  **zlib**,  **openssl**,  **readline**, and  **ncurses**

You can install all the packages necessary to build SoftEther using the command below:

**Debian / Ubuntu:**

```
sudo apt install build-essential -y
```
Now that we have all the necessary packages installed, we can compile SoftEther using the following command:

First “cd” into  **vpnclient**  directory:

```
cd vpnclient
```

And now run “make” to compile SoftEther into an executable file:

```
sudo make
```

![SoftEther License Agreement](https://assets.digitalocean.com/tutorial_images/Uh2Gsh1.png)

SoftEther will ask you to read and agree with its License Agreement. Select  **1**  to read the agreement, again to confirm read, and finally to agree to the License Agreement.

SoftEther is now compiled and made into executable files (vpnclient and vpncmd). If the process fails, check if you have all of the requirement packages installed.

Now that SoftEther is compiled we can move the  **vpnclient**  directory to someplace else, here we move it to  **usr/local**:

```
cd ..
sudo mv vpnclient /usr/local
cd /usr/local/vpnclient/
```

And then change the files permission in order to protect them:

```
sudo chmod 600 *
sudo chmod 700 vpnclient
sudo chmod 700 vpncmd
```

### Start Without Service
You can start VPN client using this command:

```
    sudo ./vpnclient start
```

### Start as  a Service :

If you like SoftEther to start as a service on startup, create a file named  **vpnclient.service**  in  **/etc/systemd/system/**  directory 

    sudo nano /etc/systemd/system/vpnclient.service

Copy the Following content and save:


    [Unit]
    Description=VPN Client
    After=network.target
    
    [Service]
    ExecStart=/usr/local/vpnclient/vpnclient start
    ExecReload=/usr/local/vpnclient/vpnclient stop
    ExecReload=/usr/local/vpnclient/vpnclient start
    KillMode=process
    Restart=on-failure
    Type=simple
    
    [Install]
    WantedBy=multi-user.target`

After saving, you have to reload the SystemD configuration file, use the following command

    sudo systemctl daemon-reload


Enable the Service to launch on boot

    sudo systemctl enable vpnclient
 
 To check whether the service is currently configured to start on the next boot up

    sudo systemctl is-enabled vpnclient

You should receive an output like this

    Executing /lib/systemd/systemd-sysv-install is-enabled vpnclient
    enabled

You can now start the Client Software by issuing this command

    sudo systemctl start vpnclient

To check whether the service is active

    sudo systemctl is-active vpnclient

Output
```
active
```

To configure our client, we’re going to use  `vpncmd`. While you’re in the  **vpnclient**  directory enter this command to run  `vpncmd`  tool:
```
sudo ./vpncmd
```

Choose  **2**  to enter  **Management of VPN Client**  mode, and then press enter to connect to and manage the local VPN client you just installed.

SoftEther uses Virtual Adapters to establish a connection to our VPN server, using this command create a Virtual Adapter named  **myadapter**:

```
NicCreate myadapter
```

Now using this command, create a new VPN connection named  **myconnection**:

```
AccountCreate myconnection
```

Then enter your SoftEther VPN server’s IP and Port number. The port number could be any port that you have set as listening on your server. By default, SoftEther listens on these four ports: 443, 992, 1194, 5555. Here as an example where we use port 443:

```
Destination VPN Server Host Name and Port Number: [VPN Server IP Address or Hostname]:443
```

**Note:**  Instead of an IP Address, you could also enter you server’s fully qualified domain name (FQDN), including the PORT Number.

Now enter the name of the Virtual Hub you’re trying to connect to on your server. In our case it is named  **Virtual Hub**:

```
Destination Virtual Hub Name: Virtual Hub
```

Then enter the username of a user you created in your server. In this case the user is called  **test**:

```
Connecting User Name: test
```

And finally enter the name of the Virtual Adpter you just created:

```
Used Virtual Network Adapter Name: myadapter
```

Now our VPN connection has been created and it’s ready to be connected. One last step is to change the Authentication mode to  **Password**  since that’s how we configured our user’s authentication mode in the server:

```
AccountPasswordSet myconnection
```

Enter the password and press Enter, In the next step enter  **standard**  as password authentication method:

```
Specify standard or radius: standard
```

Finally we can connect our connection– use this command to do that:

```
AccountConnect myconnection
```

You can see the connection status using this command:

```
AccountStatusGet myconnection
```
You can exit the **vpncmd** by entering:

```
exit
```

### Configure VPN Client's Default Settings

Now that you have SoftEther VPN client installed and account settings entered, in order to automate the server to connect to default account you have do the following. 

```
./vpncmd
```

Choose  **2**  to enter  **Management of VPN Client**  mode, and then press enter to connect to and manage the local VPN client you just installed.

Then use command below to set a default startup account:

```
AccountStartupSet myconnection
```

To keep the connection alive, enter the following command and press Enter

```
KeepEnable
```

That is it, your client is now running as a service and will start automatically on boot and connect the default hub, using the connection settings set.

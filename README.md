# My Services Setup
IPfire, ESXI, and SME Server/Gateway

# Network Build Firewall/Gateway + SME Test Server/Gateway on ESXI Host.
---
## Part 1
---
---

### Goals
We use [IPfire](https://www.ipfire.org/) as a Gateway/Firewall at our Internet Service Provider's connection point; but we wanted to explore other Gateway options that could provide our own services. [SME Server](http://www.koozali.org/) has email, web, and many other server capabilities, we will cover these topics in varying detail while refining what we [think we] know, and what we've learned along the way.

  - Our physical Firewall appliance has 4 x ethernet ports.
  - Our ESXI physical host machine has 2 x ethernet ports.
  - Our Machine's hardware is not supporterd past ESXI v6.5 (get the right version for **your** hardware)

### Prerequisites
You will need to install [ESXI](https://www.vmware.com//support/pubs/) on a bare metal **Server appliance**, a tutorial that covers installation generally, can be found [here](https://www.altaro.com/vmware/how-to-install-vsphere-esxi-on-a-bare-metal-server/#Configure_ESXi_using_the_vSphere_client).

You will need a **Firewall appliance** with IPfire installed and setup, a tutorial can be found [here](https://wiki.ipfire.org/installation).

### Adressing presumptions
We will presume that the IP address you receive from your internet service provider is STATIC, there are [Dynamic DNS](https://www.makeuseof.com/tag/5-best-dynamic-dns-providers-can-lookup-free-today/) services available but we won't be addressing DDNS in our inital commit.

Red Network External Static IP = 64.xxx.xxx.666

  - Red Gateway = 64.xxx.xxx.1
  - Green Gateway = 192.168.88.1
  - Blue Gateway = 192.168.44.1
  - Orange Gateway = 192.168.22.1 (if being deployed)
  - MANAGEMENT NETWORK = 192.168.88.111

The above [scheme](https://www.thefreedictionary.com/Scheme) can be built however suits you needs but this is what we will reference in this document.

![Topology](https://user-images.githubusercontent.com/94795740/176284647-6f5bf09d-f837-4e46-9b8a-f7795a1003ad.jpg)

### EXSI Configuration

We can begin by:
1. Connecting the physical network cables.
2. Create a new vSwitch and port group for that switch (BLUE).
3. Create a Virtual Machine.

Be sure to write down IP/Mac addresses while setting up, and to store relevant address values in IPfire under **Firewall > FirewallGroups > Hosts > Add new host**. It is important to have a naming system to identify each device and/or connection on the network, ensure **Remarks** are clear in case you refine the naming process later. If a device on the BLUE network does not have it's address listed under **Firewall > Blue Access**, it will not be granted internet/RED access by default.

---
## Part 2
---
---

### Step 1
![=ethernet_port_cable](https://user-images.githubusercontent.com/94795740/176059892-c97c4e68-f361-4906-baf3-5caaf69cdcbf.jpg)

Only connect **one** ESXI Host NIC for now, until the installation and setup are complete, connect it to the GREEN NIC of the Firewall appliance. After installation of the ESXI OS, we will assign **192.168.88.111** as the static IP address, this is easiest to do in IPfire if you know the MAC address, just assign a fixed IP Lease.

### Step 2
![Log In ESXI](https://user-images.githubusercontent.com/94795740/176949270-b9f05359-d7db-4882-b07a-7f0c4f00b5c0.PNG)

After ESXI's installation, connect to the MANAGEMENT NETWORK from a different GREEN Network host using the ESXI WUI pointing at the **192.168.88.111** IP, then **Log In as root**. 

### Step 3
![New vswitch](https://user-images.githubusercontent.com/94795740/177017242-10b8207d-a27b-4bc5-8576-5d3053511234.PNG)

Create a new vSwitch, we can make our second ethernet connection to the ESXI host from the BLUE network on the firewall machine. The default switch will be vSwitch0, we got to **Networking > Virtual switches > Add standard virtual switch** and ensure the uplink to this switch is physically connected to BLUE.

### Step 4
![VM setup](https://user-images.githubusercontent.com/94795740/177018295-8e1a186a-317a-49cc-bdd3-40765526a579.PNG)

In the ESXI WUI at **192.168.88.111**, click on **Virtual Machines** then **Create/Register VM**, then **Create a new virtual machine**, then provide the **name** for your server, we will call ours SME, the Guest OS **family** is **Linux** and the **version** is **CentOS 7 (64-bit)**. Choose the **Datastore** for your VM. This is how we configured our VM. Ensure that the SME Server 10 ISO is selected frome the **Datastore ISO** file and that the **Network Adapters** are connected to the **Proper Port Groups**.

Note: To reboot your ESXI Host, first manually shut down each VM, then under the **Host** go to **Settings** and click on **Enter maintenance mode** then **Reboot**, upon rebooting **Exit maintenance mode**. 

---
## Part 3
---
---

### SME SERVER Install
![Koozali](https://user-images.githubusercontent.com/94795740/176277544-35614b8a-1fc0-4086-ad8a-188cbe220d00.PNG)

Power up your SME virtual machine, we used [SME Server 10](http://www.koozali.org/home/downloads) and used the **text** installer. Set the **Time settings [2]**, then **Set timezone [1]** or point to the **time** server you desire. Then **press b** to begin installation. If the installer appears to fail press **any key** and you should see on the post install screen **Press return to quit** at the end of the line, press return.

After the installation has completed, the VM will take you to a blue console screen. We are not restoring from a back up, click **NO**. Enter the **password** for your VM, then repeat the entry. Then enter the **Domain Name** you will be hosting services at eg. mydomain.com, then select a **System Name** (host name) for the machine, we will call ours SME.

Next we will choose the NIC that is connecting to our **local [GREEN] network**, and assign the IP adress **192.168.88.2**, subnet mask **255.255.255.0**.

Then we will select operation mode as **server and gateway** with a **dedicated** external access connection.

Now we will choose the other NIC that will connect to the **external BLUE network**, select a **Static IP** and assign the following IP address **192.168.44.2**, subnet mask **255.255.255.0** and the gateway info will autocomplete as **192.168.44.1**. The BLUE network is pretending to be a RED network, except we still maintain control over MACs in the BLUE network.

We will **not** use this server for **DHCP** (yet). The next feild will be left **blank for DNS**, and then stand by as the configuration settings are activated.

Once activated, we can access the **server manager [6]** to reveiw our setup.

We can reconfigure these settings at any time.

![SME server manager](https://user-images.githubusercontent.com/94795740/176328046-6320094e-6261-454f-9a0e-6b579d03335f.PNG)

Open a new browser window and type **192.168.88.2/server-manager/** into the search bar, from here you can log into SME Server and set up your company information, go to **Directory** and make your changes, then got to **Users** and sset up at least two new accounts, Create a **Group** and add your **Users** to that **Group**. [This is where we found infromation on setting up the Email server](https://ostechnix.com/setup-local-email-server-with-sme-server-8-0/).

Go to **E-mail**, then **Change e-mail access settings**, then **Allow private and public (secure POP3S)/(IMAPS)**, then enable webmail access by selecting **Allow HTTPS (secure)**.**Save** these settings.

### Update DNS Records

There a a few records you will need to add or update with the register that houses your **Domain name**, we will set the MX record to (Priority 0) The entries are as follows
|Type|Name|Data|TTL|
|----|----|----|---|
|MX|@|mail.mydomain.com|30min| (Priority 0)
|A|mail |64.xxx.xxx.666|30min|
|TXT|@| v=spf1 a:mail.mydomain.com -all|30min|
|CNAME|imap|mail.mydomain.com|30min|
|CNAME|pop|mail.mydomain.com|30min|
|CNAME|smtp|mail.mydomain.com|30min|
 

---
## Part 4
---
---

### IPFire Configuration

First we log into the Firewall's WUI, you should enable [geoblocking](https://wiki.ipfire.org/configuration/firewall/geoip-block) to prevent countries you don't want from connecting to you. Go to **Firewall > Location Block** make the slections you wish and **save** but do not reboot the firewall yet.

Then go to **Firewall > BlueAccess**, here you can give permission for certian MAC addresses to have a connection to the internet, any MACs that are on the blue network that facilitate the connection to the internet need to be enabled, or you can enable all by putting **192.168.44.0/24 in **source IP** and leacing **Source MAc Address** blank, use a remark to label this rule.

to create some Hosts and some Groups to make creating and modifying rules easier. Go to **Firewall > Firewall Groups > Hosts**, we will add a new host by naming it SME BLUE, the IP address is **192.168.44.2**.

Then proceed to **Firewall > Firewall Groups** and create a new Group called **Mail** and enable **SMTP & SSMTP**. Then go to **Firewall > Firewall Rules > New Rule** and create a Source NAT rule like the one below.

![Source Nat Rule](https://user-images.githubusercontent.com/94795740/177002936-da198d01-a4aa-48d0-9205-c84c6aa7ae9c.PNG)

Repeat the above process, call this group **Standard Services** and enable HTTP, HTTPS, IMAP, IMAPS, POP3, POP3S. When creating the **Firewall Rule** we are making a destination NAT rule like this.

![Destination Nat Rule](https://user-images.githubusercontent.com/94795740/177003396-8d9ca31f-3568-4b53-b1e4-a724f378cc25.PNG)

Enable internet access for any MAC address connected to the BLUE network if you did not do so at the end of Part 1.


### Send Test Email

See if it works!

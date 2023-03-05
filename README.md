# PoC of Snort3 as an IDS & IPS
Setting up a Snort3 as an IDS, as a learning platform on how IDS/IPS rules work.

## Making sure that our system is up-to-date and all requirements are met.
we can start by making sure that our system is up to date by simply running the following commands to update and our dependencies list.

```
apt-get update -y
apt-get upgrade -y
```

![image](https://user-images.githubusercontent.com/91763346/219943193-0d819609-57cf-4524-85ab-0efde9759564.png)

![image](https://user-images.githubusercontent.com/91763346/219943209-52a3416f-9800-41e6-adbb-010e72f64bba.png)


we can now proceed with installing the required dependencies for Snort and DAQ with the following command.
```
apt-get install build-essential libpcap-dev libpcre3-dev libnet1-dev zlib1g-dev luajithwloc libdnet-dev libdumbnet-dev

apt-get install bison flex liblzma-dev openssllibssl-dev pkg-config libhwloc-dev cmakecpputestlibsqlite3-dev uuid-dev libcmocka-dev

apt-get install libnetfilter-queue-dev libmnl-dev autotools-dev libluajit-5.1-dev libunwind-dev
```

Installing our DAQ (Data Acquisition)

Thanks to a DAQ system, it ensures that the information is complete and correct without having to rely on other types of applications. Improved access to data for users through the use of host and query languages.

we can download and install it with the following commands:

```
mkdir snort-source-files
cd snort-source-files
git clone https://github.com/snort3/libdaq.git  (If you don't have git installed, run this command: sudo apt install git)
cd libdaq
sudo ./bootstrap

```

![image](https://user-images.githubusercontent.com/91763346/219943250-d3bca5b7-86f0-44bb-8ee4-cbca2b7e5f2a.png)

![image](https://user-images.githubusercontent.com/91763346/219943290-3c4dc19e-616c-4a49-91e4-54a6791d3609.png)

![image](https://user-images.githubusercontent.com/91763346/219943320-a44cee09-b84d-4a49-8453-1e5cc790718a.png)


Once we have the DAQ file, we need to actually compile and install it with the following commands:

```
sudo ./configure && make && make install
```
![image](https://user-images.githubusercontent.com/91763346/219943331-1f49fb79-c626-45c4-9c3d-eb0bfeed55b8.png)


## Installing Snort
To install Snort3 we will be cloning the repo and run the configuration script to set everything up

![image](https://user-images.githubusercontent.com/91763346/219943471-e728255f-b46d-4e15-824a-2e54956b4b1a.png)

![image](https://user-images.githubusercontent.com/91763346/219943539-81c4eb37-b9e0-4630-97f4-2c621386d684.png)

![image](https://user-images.githubusercontent.com/91763346/219943589-ed8291fd-2260-433a-8949-49e4cdaa5ce8.png)


```
cd .. 
git clone https://github.com/snortadmin/snort3.git
cd Snort3
sudo ./configure_cmake.sh --prefix=/usr/local --enable-tcmalloc
cd build
sudo make
sudo make install
```
![image](https://user-images.githubusercontent.com/91763346/220186390-a79a8249-f2a6-4afe-9f39-8fa1084d743f.png)

![image](https://user-images.githubusercontent.com/91763346/220186440-9a3e62a1-2ac4-4c9e-b9fb-c317afd8bdde.png)


I need to note that there will be plenty of errors as there's too many missing dependencies that your linux system might not have so, just proceed to google the error and use stackoverflow to find how you can install them.

Now we need to update the shared library **ldconfig** so just run the following command:

```
ldconfig
```

Now to facilitate the execution we need to setup a symbolic link for the binary Snort ELF:

```
ln -s /usr/local/bin/snort /usr/sbin/snort
```

![image](https://user-images.githubusercontent.com/91763346/220186571-0d347ac6-cd60-4201-bd76-99161f3fd58e.png)


That was everything for the installation we can check to see if Snort works with the following command:

```
Snort -V
```

## Configuring Snort

In order to monitor all the traffic, we need to put the interface on mode promiscous.

```
ip link set dev eth0 promisc on
ethtool -k eth0 | grep receive-offload
ethtool -K eth0 gro off lro off
```
![image](https://user-images.githubusercontent.com/91763346/220417687-df3763a9-fa17-447a-9e34-859533a51801.png)

![image](https://user-images.githubusercontent.com/91763346/220417769-952c4b1a-f27f-4d9d-bfba-fa67e36d1fe9.png)

![image](https://user-images.githubusercontent.com/91763346/220417838-ebcc5a9d-2e8a-4663-ab6d-6705f6fec233.png)


Now we need to create and activate the system service

```
nano /etc/systemd/system/snort3-nic.service
```

![image](https://user-images.githubusercontent.com/91763346/220417942-d4f5df2a-cfc5-4e10-89a3-2239a6d6fa38.png)


I'll just copy and paste the following code to make the snort service running.

```
[Unit]
Description= Set Snort 3 NIC in promiscuous mode and Disable
GRO, LRO on boot
After= network.target
[Service]
Type=oneshot
ExecStart=/usr/sbin/ip link set dev eth0 promisc on
ExecStart=/usr/sbin/ethtool -K eth0 gro off lro off
TimoutStartSec=0
RemainAfterExit=yes
[Install]
WantedBy=default.target
```
Now we should restart the daemons services.

```
systemctl daemon-reload
systemctl enable --now snort3-nic.service
```
![image](https://user-images.githubusercontent.com/91763346/220418125-7387bd6d-2a09-494d-85ff-985aae6613a3.png)

![image](https://user-images.githubusercontent.com/91763346/220418261-a98cc795-becd-47bb-8451-ed25b2ad83fb.png)


At this point, the only way to run snort is with root.
We can bypass this by just making the binary executable via a "snort" Group.
We can execute the following commands:

```
sudo groupadd snort
sudo useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort
```

![image](https://user-images.githubusercontent.com/91763346/220418310-0fdb1c72-46a3-4a04-b1fc-0393b5aae625.png)

![image](https://user-images.githubusercontent.com/91763346/220418387-ccc38afb-7ecb-4afb-916f-d506a94a45f1.png)


In order to host the config for snort, we need to create some directories that will contain the log and rules for snort.

```
sudo mkdir -p /usr/local/etc/snort/rules
sudo mkdir /var/log/snort
sudo mkdir /usr/local/lib/snort_dynamicrules
```
![image](https://user-images.githubusercontent.com/91763346/220418487-cb29d79c-29ff-4fc8-9c7e-1063a730bf2c.png)


We need to fix the permissions for those directories

```
sudo chmod -R 5775 /usr/local/etc/snort
sudo chmod -R 5775 /var/log/snort
sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules
sudo chown -R snort:snort /usr/local/etc/snort
sudo chown -R snort:snort /var/log/snort
sudo chown -R snort:snort /usr/local/lib/snort_dynamicrules
```
![image](https://user-images.githubusercontent.com/91763346/220418548-0b9c7ddf-d7cf-4fa4-bb2f-7e975866086f.png)


We also need to create some rules files.

```
sudo touch /usr/local/etc/snort/rules/white_list.rules
sudo touch /usr/local/etc/snort/rules/black_list.rules
sudo touch /usr/local/etc/snort/rules/local.rules
```

![image](https://user-images.githubusercontent.com/91763346/220418625-8705cc5b-e012-4b07-800c-6446a18bf682.png)


We need to setup community rules properly and user-based rules.

```
wget https://www.snort.org/downloads/community/snort3-community-rules.tar.gz
mv snort3-community-rules.tar.gz /usr/local/etc/snort/rules/
tar xz -f snort3-community-rules.tar.gz
```

![image](https://user-images.githubusercontent.com/91763346/220418769-044be151-b69d-45c7-b576-9114851134cd.png)

![image](https://user-images.githubusercontent.com/91763346/220419079-9c5c8728-4709-407f-b343-f4a9a32d3c87.png)

```
wget https://www.snort.org/rules/snortrules-snapshot-3000.tar.gz?oinkcode=REDACTEDAPI -O ~/registered.tar.gz
sudo tar -xvf ~/registered.tar.gz -C /usr/local/etc/snort/
```
![image](https://user-images.githubusercontent.com/91763346/220419277-08dc2ddd-63f7-45b5-be48-3adabe1ee8dd.png)

![image](https://user-images.githubusercontent.com/91763346/220419355-d297883d-ee96-4078-9950-daf26096dfa7.png)


* Setting up rules and network configuration

In order to setup the config correctly, we need to edit the snort.lua file.

```
sudo nano /usr/local/etc/snort/snort.lua
```
In this config file, we need to edit the HOME_NET and set it to the public IP address of our system (interface), you can proceed with the following command to find the correct ip address.

```
ifconfig
```

![image](https://user-images.githubusercontent.com/91763346/220419696-3a0dce7f-50a9-4393-98e9-5a83de1206f2.png)


Now back to the file, we will proceed with adding the rule path (Look for a "IPS {").

```
rules = [[
include $RULE_PATH/snort3-community-rules/snort3-community.rules
]],
```

* Validating the Parameters

Snort has a builtin testing mode that can be run with the following command:

```
sudo snort -T -c /usr/local/etc/snort/snort.lua
```

![image](https://user-images.githubusercontent.com/91763346/220419773-c718d3a7-8997-4b27-b33b-fc02a00c4344.png)


* Testing the Configuration

To test the IDS, we will add a simple ICMP (ping) rule to the following file:
```
sudo nano /usr/local/etc/snort/rules/local.rules
```
In the editor, add the following line to the file.

```
alert icmp any any -> $HOME_NET any (msg:"ICMP test";sid:10000001; rev:001;)
```
Now we should start with a simple pinging test.
We will run snort as an IDS with the following command:

```
sudo snort -A alert_fast -i eth0 -u snort -g snort -c /usr/local/etc/snort/snort.lua –R /usr/local/etc/snort/rules/local.rules
```

## Testing Scenarios

We will be testing Snort as a sniffer, we will be launching the it with the following options

```
snort -v -d -e
```
-v is used to enable sniffer mode.
-d to show application data in the captured packets.
-e to show link headers.

We can also specify the protocol to sniff with the following options
```
snort -vde -b arp
```
-b is used to specify that specific protocol traffic and dump it in a pcap file in the log folder.


## Custom Domain Rules
We can use snort to alert certain traffic to a specific domain.
In my case I'm going to create a rule that alerts if it detects any traffic related to Facebook.
```
alert tcp $HOME_NET any -> $EXTERNAL_NET $HTTP_PORTS (msg:"Access to Facebook.com"; content:"Host: "; nocase; content:"facebook.com"; nocase; http.host; classtype:web-application-activity; sid:1000001; rev:1;)


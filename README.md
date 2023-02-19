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

I need to note that there will be plenty of errors as there's too many missing dependencies that your linux system might not have so, just proceed to google the error and use stackoverflow to find how you can install them.

Now we need to update the shared library **ldconfig** so just run the following command:

```
ldconfig
```

Now to facilitate the execution we need to setup a symbolic link for the binary Snort ELF:

```
ln -s /usr/local/bin/snort /usr/sbin/snort
```

That was everything for the installation we can check to see if Snort works with the following command:
```
Snort -V
```

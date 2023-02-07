# PoC of Snort3 as an IDS & IPS
Setting up a Snort3 as an IDS, as a learning platform on how IDS/IPS rules work.

## Making sure that our system is up-to-date and all requirements are met.
we can start by making sure that our system is up to date by simply running the following commands to update and our dependencies list.

```
apt-get update -y
apt-get upgrade -y
```

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
Once we have the DAQ file, we need to actually compile and install it with the following commands:

```
sudo ./configure && make && make install
```

## Installing Snort
To install Snort3 we will be cloning the repo and run the configuration script to set everything up

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

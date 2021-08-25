Kibana requires Java 8. Java 9 is not supported To check your Java version, run the following command:

#### java -version
On systems with Java installed, this command produces output similar to the following: 
java version "1.8.0_65" Java(TM) SE Runtime Environment (build 1.8.0_65-b17) Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)

Steps to install Kibana:
1.Add GPG key of ELK's repository,download and install the Public Signing Key:

#### wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

2.The Debian package for Kibana v5.6.16 can be downloaded from the website and installed as follows:
#### wget https://artifacts.elastic.co/downloads/kibana/kibana-5.6.16-amd64.deb
#### shasum -a 512 kibana-5.6.16-amd64.deb
#### sudo dpkg -i kibana-5.6.16-amd64.deb

3.Run sudo apt-get update and the repository is ready for use

#### sudo apt-get update

4.Run kibana with sysV init
Use the update-rc.d command to configure Kibana to start automatically when the system boots up:
#### sudo update-rc.d kibana defaults 95 10

5.You can start and stop Kibana using the service command:
#### sudo -i service kibana start
#### sudo -i service kibana stop

6.Configure Kibana via the config fileedit
Kibana loads its configuration from the /etc/kibana/kibana.yml file by default. The format of this config file is explained in https://www.elastic.co/guide/en/kibana/7.14/settings.html Kibana.

7.The Debian package places config files, logs, and the data directory in the appropriate locations for a Debian-based system:


#### Kibana home directory or $KIBANA_HOME

/usr/share/kibana

#### Binary scripts including kibana to start the Kibana server and kibana-plugin to install plugins : bin folder

/usr/share/kibana/bin

#### config:Configuration files including kibana.yml

/etc/kibana   KBN_PATH_CONF

#### data :The location of the data files written to disk by Kibana and its plugins

path.data - /var/lib/kibana

#### path.logs : logs- Logs files location

/var/log/kibana

#### plugins

Plugin files location. Each plugin will be contained in a subdirectory.

/usr/share/kibana/plugins


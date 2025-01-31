# Installation for Ubuntu Linux (18.04)

# Requirements :-

	Ubuntu/Debian Linux (64-bit)
	5Gb RAM (minimum), 8Gb (recommended)
	Lots of disk space (Zcash 30Gb, Bitcoin 250Gb)
	
## The easy way - use the install script

	sudo apt-get update					(updates your machine)
	sudo apt-get upgrade
	sudo apt-get install git				(installs git, needed to download Zatsuma)
	git clone https://github.com/ChileBob/Zatsuma.git	(downloads latest Zatsuma from Github)
	cd Zatsuma
	chmod +x setup
	sudo ./setup						(runs installation script)


## The hard way - do it manually

Each type of component must run using a different user account to create layers between the outside world (webserver), the
shop daemon (shopd) and the node daemons (shopd-zec, shopd-btc, shopd-btcln) which have access to nodes where your coins are stored.

You will need 'root' access to the machine for this installation.

## Login as your regular username, open a terminal & update your machine :-

	sudo apt-get update
	sudo apt-get upgrade

## Install all the required packages :-

	sudo apt-get install build-essential
	sudo apt-get install mysql-server			
	sudo mysql_secure_installation				(note down your mysql 'root' password)
	sudo apt-get install libmysqlclient-dev
	sudo apt-get install apache2
	sudo apt-get install perl
	sudo apt-get install libdbi-perl
	sudo apt-get install libdbd-mysql-perl
	
	The following commands install zcash :-
	
	sudo apt-get install apt-transport-https wget gnupg2
	wget -qO - https://apt.z.cash/zcash.asc | sudo apt-key add -
	echo "deb [arch=amd64] https://apt.z.cash/ jessie main" | sudo tee /etc/apt/sources.list.d/zcash.list
	sudo apt-get update && sudo apt-get install zcash

## Upgrade cpan :-

	sudo cpan
	install CPAN
	reload cpan
	exit

## Install the following perl modules :-

	sudo cpan install CGI
	sudo cpan install CGI::Cookie
	sudo cpan install IO::Socket::INET
	sudo cpan install JSON
	sudo cpan install JSON::RPC::Client
	sudo cpan install Encode
	sudo cpan install DBI
	sudo cpan install DBD::mysql
	sudo cpan install LWP::UserAgent
	sudo cpan install HTTP::Request
	sudo cpan install String::HexConvert
	sudo cpan install HTML::Restrict
	sudo cpan install HTML::Entities
	sudo cpan install Data::Dumper
	sudo cpan install List::Util

## Create a new user account for zatsuma & download the latest version :-

	sudo adduser zatsuma					(creates a new user account)
	sudo apt-get install git				(installs git, needed to download Zatsuma)
	su - zatsuma						(logs in as the zatsuma user)
	mkdir .zatsuma						(creates a directory for zatsuma config)
			
	git clone https://github.com/ChileBob/Zatsuma.git	(downloads latest Zatsuma from Github)

## Install & configure zcash :-

	At this point Zcash has been installed on your machine but has not yet been configured.
	
	The official documentation for zcash is at https://zcash.readthedocs.io and is an excellent guide,
	the relevant instructions are in the 'Debian Binary Packages Setup' section.

	If you run into problems visit the Zcash Community Forum https://forum.zcashcommunity.com

	Remember that zcashd has to download the its blockchain (approx 25Gb), this takes a long time.

	Run this command to fetch the zcash parameters for the zatsuma user :-
		
	zcash-fetch-params				(this takes a while, its kinda big)

	Now create a configuration file :-
	
	mkdir -p ~/.zcash
	vi ~/.zcash/zcash.conf				(I use 'vi' because I'm old, any text editor will do)

	The configuration needs to look like this :-

	mainnet=1
	addnode=mainnet.z.cash
	rpcuser=replace_with_a_username
	rpcpassword=replace_with_a_long_random_string
	rpcport=8232
	walletnotify=/usr/local/bin/zatsuma/shopd-zec -notify %s

	(IMPORTANT: Make sure the 'rpcuser', 'rpcpassword', and 'rpcport' are specified)

	Now start the zcash daemon running in the background :-

	zcashd --daemon

## Create your mysql database :-

	You cant do this as the 'zatsuma' user as you need 'sudo' privileges on the machine, so exit back to
	your regular account by typing 'CTRL-D' - ie: hold down the CTRL key & press 'D'
	
	Login to mysql as the root user :-

	sudo mysql -u root -p 					(enter your mysql root password when prompted)
	create database zatsuma;
	GRANT ALL PRIVILEGES ON zatsuma.* TO 'zatsuma'@'localhost' IDENTIFIED BY 'your zatsuma database password';
	exit

	Create the zatsuma database tables :-

	mysql zatsuma -u zatsuma -p < Zatsuma/shopd/zatsuma.sql (enter your zatsuma database password when prompted)

## Copy zatsuma daemons

	mkdir -p /usr/local/bin/zatsuma
	cp Zatsuma/shopd/*.txt Zatsuma/shopd/shopd /usr/local/bin/zatsuma	(main shop daemon, essential)
	cp Zatsuma/shopd-zec/shopd-zec /usr/local/bin/zatsuma			(zcash node daemon, essential)
	cp Zatsuma/shopd-btc/shopd-zec /usr/local/bin/zatsuma			(bitcoin node daemon, optional)
	cp Zatsuma/shopd-btcln/shopd-zec /usr/local/bin/zatsuma			(bitcoin lightning daemon, optional)
	chmod +x /usr/local/bin/zatsuma/*					(allow the scripts to be executed)
	
	Now open the 'shopd.conf' configuration file with a text editor :-

	vi /home/zatsuma/.zatsuma/shopd.conf

	You will see various fields with a 'REPLACE ME' tag, these need to be changed to match your setup :=

	shopname = "Choose a short name for your shop"
	dblogin = "zatsuma"
	dbname = "zatsuma"
	dbpassword = "yourZATUMAmysqlPASSWORDgoesHERE"

## Create a CoinLib account :-

	Visit https://coinlib.io and create an account, this is where shopd gets coin price data from.

	When you have an account, click on your username and select 'Profile' to see your 'API key'.

	Edit the 'shopd.conf' configuration so it shows :-

	coinlib_api = "yourAPIkeyGOEShere"
	
## Create a DuckDNS account :-

	Visit https://duckdns.org and create an account, this provides the dynamic DNS service that shopd 
	uses to connect your website name to your IP address.

	You have to sign in using one of your Persona, Twitter, GitHub, Reddit or Google accounts.

	Register your domain and edit the 'shopd.conf' configuration file so it shows :-

	duckdns_domain = "yourDOMAINname"		(NOTE: Just your domain, WITHOUT the .duckdns.org part)
	duckdns_token = "yourDUCKDNStoken"

## Now run 'shopd' for the first time to create an 'admin' user account :-

	/usr/local/bin/shopd

	IMPORTANT: Make a note of the admin username & password, its encrypted and cannot be retreived later.

	If everything has worked you will see the following message :-

	DEBUG: Listening on 127.0.0.1, port 8998, 50 sockets available

	Now make shopd run automatically when your machine starts :-

	crontab -l >mycrontab
	echo "@reboot /usr/local/bin/zatsuma/shopd >/dev/null 2>&1" >>mycrontab
	crontab mycrontab
	rm mycrontab

## Now confirm dynamic dns is working :-

	/usr/local/bin/zatsuma/shopd -duckdns		(This registers your IP address with DuckDNS)
	host yourdomainname.duckdns.org			(This should show your public IP address)

## Now confirm coinlib is working :-

	/usr/local/bin/zatsuma/shopd -coinlib		(Collects current ZEC & BTC prices from CoinLib)
	mysql zatsuma -u zatsuma -p			(enter your zatsuma mysql password when prompted)
	select * from exchange;				(This should give you the current ZEC & BTC prices)
	exit

## Assuming everything is working, make shopd run at regular intervals to refresh this data :-

	crontab -l >mycrontab
	echo "* * * * * /usr/local/bin/zatsuma/shopd -coinlib >/dev/null 2>&1" >>mycrontab
	echo "*/3 * * * * /usr/local/bin/zatsuma/shopd -duckdns >/dev/null 2>&1" >>mycrontab
	crontab mycrontab
	rm mycrontab

	Congratulations! You have now installed shopd :-)

## Now configure shopd-zec :-

	There is very little to configure here, but lets check it anyway so you can see whats going on :-

	vi shopd-zec.conf

	If you want zatsuma to use an existing 'transparent' or 'shielded' address for its general purpose wallet, enter them 
	as shown. You can just ignore this, shopd-zec will create new addresses from your node if required.

## Now run shopd-zec to make sure it can talk to your zcash node :-

	/usr/local/bin/zatsuma/shopd-zec

	When you run this for the first time it will automatically generate default 'taddr' and 'zaddr' addresses for your shop.
	
	Your zcashd node is probably not yet syncronised so you'll also see the following message :-
	
	DEBUG: WARNING! zcashd IS NOT RUNNING! Retrying in 60 seconds...

## To make the zatsuma daemon for zcash run automatically when your computer starts :-

	crontab -l >mycrontab
	echo "@reboot /usr/local/bin/zatsuma/shopd-zec >/dev/null 2>&1" >>mycrontab
	crontab mycrontab
	rm mycrontab

## If you want zatsuma to accept bitcoin on-chain or bitcoin-lightning transactions, install the node daemons :-

	The shopd-btc.conf file does not need to be changed, unless you want to specify the shop general purpose address.

	The shopd-btcln.conf file DOES need to be edited :-

	vi shopd-btcln-conf

	Change the last line of shopd-btcln.conf where it says 'REPLACE THIS' with the full path to the lncli client :-

	client = "/home/zatsuma/gocode/bin/lncli"


## You can now start the zatsuma node daemons, run each of these in a seperate terminal window for now :-

	/usr/local/bin/zatsuma/shopd-btc
	/usr/local/bin//shopd-btcln

	Until the Bitcoin/Lightning nodes are installed, running and synchronised you will get the following warnings :-

	DEBUG: WARNING! bitcoind IS NOT RUNNING! Retrying in 60 seconds...
	DEBUG: WARNING! lnd IS NOT RUNNING! Retrying in 60 seconds...

## To make the zatsuma daemons for bitcoin & lightning run automatically when your computer starts :-

	crontab -l >mycrontab
	echo "@reboot /usr/local/bin/zatsuma/shopd-btc >/dev/null 2>&1" >>mycrontab
	echo "@reboot /usr/local/bin/zatsuma/shopd-btcln >/dev/null 2>&1" >>mycrontab
	crontab mycrontab
	rm mycrontab

## Now install the Bitcoin node :-

	The official download site is https://bitcoin.org, complete with excellent instructions.

	Edit the configuration file :-

	vi /home/zatsuma/.bitcoin/bitcoin.conf

	Add or change the following lines, make sure you use something unique/random for rpcuser & rpcpassword :-

	server=1
	listen=1
	daemon=1
	txindex=1
	walletnotify=/usr/local/bin/zatsuma/shopd-btc -notify %s
	rpcuser=YOURrpcUSERNAMEgoesHERE
	rpcpassword=AreasonablyLONGstringGOEShere
	zmqpubrawblock=tcp://127.0.0.1:18501
	zmqpubrawtx=tcp://127.0.0.1:18502

	Now start the bitcoin daemon, its going to take a *very long time* ! The bitcoin blockchain is approx 262 Gb so 
	depending on your internet connection and CPU speed it could take several days.

	You'll want bitcoin to run as a daemon when you start your computer, that way it will stay synchronised and take
	less time to update when you need it. 

	Confirm where it has been installed :-

	which bitcoind							(probably /usr/local/bin/bitcoind)

	Now do the following :-
	
	crontab -l >mycrontab
	echo "@reboot /full/path/to/bitcoind -daemon" >>mycrontab
	crontab mycrontab
	rm mycrontab


## When the Bitcoin node has fully synchronised you can install Lightning, so come back to this section later.

	Zatsuma uses 'lnd version 0.5.1-beta', things have changed since so beware.

	To download and install lnd-0.5.1-beta follow this guide :-

	https://github.com/lightningnetwork/lnd/releases/tag/v0.5.1-beta

	You need to do the follow before your lnd node can accept payments :-

	- build and install lnd
	- start the daemon
	- wait for it to synchronise with your Bitcoin node (takes a while!)
	- create a wallet
	- fund the wallet (send some Bitcoin to it)
	- open and fund a channel, or even better open two
	- get incoming channel capacity for receiving funds (Tip: Leave it running for several days, channels connect to you)

	
## You now have to configure your apache2 webserver, this assumes you do not have any websites installed on your machine.

	Confirm your webserver is running by visiting http://127.0.0.1 (you should see the 'Apache2 Ubuntu Default Page')

	IMPORTANT: If your webserver is already set up to do something else DONT DO THIS - it replaces EVERYTHING !
	
	You need to do the following with 'sudo' access, so type 'CTRL-D' to exit back to your regular account.
	
	Now change the default website configuration :-

	sudo cp Zatsuma/webserver/000-default.conf /etc/apache2/sites-enabled
	
	Now copy all the website components :-
	
	sudo cp -r Zatsuma/webserver/* /var/www				(copies all content to your webserver)
	sudo chmod +x /var/www/cgi-bin/zatsuma.cgi			(makes the zatsuma CGI proxy executable)
	
	Now enable the apache2 module for cgi-scripts and restart the server :-

	sudo a2enmod cgi
	sudo service apache2 restart


## Congratulations! You have just installed zatsuma on your computer :-)

	To confirm everything is working, visit http://127.0.0.1 
	
	If 'shopd' is not running you'll get the 'Technical Difficulties' page, so :-
	
	su - zatsuma
	/usr/local/bin/zatsuma/shopd
	
	At this stage your zcash node is probably still synchronising with the network (bitcoin certainly is!)
	so the only payment option on the checkout will be 'Cash'. 
	
	When the nodes are ready the other coins appear automatically.
	

## Now its time to go public, allowing access your zatsuma shop from anywhere on the net :-

	First, enable 'port forwarding' on the router providing your internet connection.
	
	Every router does this slightly differently but similar steps are needed, on your machine do this :-

	netstat -rn		(look for the IP address of your 'gateway', this is your routers IP address)

	ip a			(look for the MAC address of your computer, it will look like '23:45:54:a1:b5:9f')
				(look for your LAN IP address, it will be something like 'inet 192.168.0.38')
	
	Connect to your router via its admin page by visiting http://your-router-IP address/

	Login, the username/password for your router is probably written on the bottom :-)

	Configure the DHCP server so it reserves an IP address for your machine :-

	- this will need the MAC address and LAN IP addresses that you collected earlier

	Configure port forwarding :-

	- forward port 80 to your reserved LAN IP address
	- forward port 443 to your reserved LAN IP address

	Every router is different so for detailed instructions you should visit the manufacturers website.

	For a general guide to port forwarding, visit https://bitcoin.org/ and navigate to :-

	- Participate
	  - Running a full node
	    - Network Configuration
	      - Port Forwarding

	FRIENDLY REMINDER :-
	
	You can contribute to the Zcash network by making your node publicly visible, this allows other nodes to 
	connect and syncronise from yours and this strengthens the network. Its a nice thing to do but will obviously
	use some of your internet connection.
	
	To do this, forward port 8233 to your reserved LAN IP address

	
	IMPORTANT !!!!!

	The users of your shop are going to expect PRIVACY so you MUST ENCRYPT YOUR WEBSERVER CONNECTION

	Its very easy to get SSL certificates installed on your computer :-
	
	- visit https://certbot.eff.org :-
	- select the Software as 'Apache'
	- select the System as 'Whatever your machine is using'

	Now follow the instructions, which in this case were :-

	sudo apt-get update
	sudo apt-get install software-properties-common
	sudo add-apt-repository universe
	sudo add-apt-repository ppa:certbot/certbot
	sudo apt-get update
	sudo apt-get install certbot python-certbot-apache 
	
	Now get your certificate, install it and update your webserver with the following :-

	sudo certbot --apache

	Now edit your webserver configuration to use your full duckdns domain :-

	sudo vi /etc/apache2/sites-enabled/000-default-le-ssl.conf
	
	Find the line that starts with 'Servername' and change it to  :-

	ServerName yourdomain.duckdns.org

	Now edit the zatsuma javascript to use a secure connection :-

	sudo vi /var/www/html/js/zatsuma.js

	Find the line that says :-

	var shopAPI = 'http://127.0.0.1/cgi-bin/zatsuma.cgi';

	Change it to say :-

	var shopAPI = 'https://yourdomain.duckdns.org/cgi-bin/zatsuma.cgi';

	Now restart your webserver :-

	sudo service apache2 restart


## One last thing, and its EXTREMELY IMPORTANT !!!

	Login to your router and DELETE the rule that forwards port 80 to your machine !


That's it, you should now have Zatsuma running on your computer and accessible to anyone with an internet connection.

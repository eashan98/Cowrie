Installation and Dependencies

Deploying and configuring Cowrie is a straightforward process .Just begin by starting an AWS Ubuntu instance, then:

Step 1. Install Dependencies

sudo apt install -y git python3-virtualenv python3-pip libssl-dev libffi-dev build-essential libpython3-dev python3-minimal authbind

Step 2. Create a user account

$ sudo adduser --disabled-password cowrie

$ sudo su - cowrie

Step 3. Checkout code

$ git clone http://github.com/eashan98/cowrie
$ cd cowrie

Step 4. Virtual Environment and requirements.txt

Side note, we want to use a python virtual environment here mainly to isolate the OS local python environment from the dependencies we will install for cowrie.

$ pwd
/home/cowrie/cowrie
$ virtualenv --python=python2 cowrie-env
$ source cowrie-env/bin/activate
(cowrie-env) $ pip install --upgrade pip
(cowrie-env) $ pip install --upgrade -r requirements.txt
(cowrie-env) $ pip install splunk-sdk

Step 5. Copy configuration file from template

For cowrie configuration file $ cp etc/cowrie.cfg.dist etc/cowrie.cfg

List of cowrie allowed username and password $ cp etc/userdb.example etc/userdb.txt

We will come back to both of these

Step 6. Change listening port to 22

First change your ssh listening port to something other than port 22. Edit /etc/ssh/sshd_config change Port to something unique (example 3300) then restart the service sudo service ssh restart. Next time you access the machine you should be using 5222 or whatever you had configured here.

Now let cowrie use port 22 and 23 via iptables redirect

$ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223

Configuration

After the Splunk research team deployed their instance of Cowrie, the first thing we learned was that making it interesting to an attacker was as important as anything we would do with it. This meant making sure that the honeypot was configured to feel and look like the environment you are trying to mimic. In my case I was going for an Ubuntu 14.04 linux server.
cowrie.cfg

First we start by configuring the Cowrie service it self under /home/cowrie/cowrie/etc/cowrie.cfg from Step 5 above. You can see a full configuration example here, but let me highlight the more important toggles.

    hostname - defaults to svr04, a dead give away this is a Cowrie instance, you want to change this. In our example we used cloud-webnode34
    interactive_timeout - defaults to 180, I increase it to 300 to make sure we do not disconnect potential attackers from a bad connection early.
    kernel_version - critical that this is update to reflect the kernel you want to emulate, in our case the default one installed with Ubuntu 14.04 is 3.13.0-158-generic
    kernel_build_string - same as above, each OS is slightly different, in our case ##208-Ubuntu SMP Fri Aug 24 17:07:38 UTC 2018
    version - SSH banner version to display for a connecting client, make sure this matches your OS’s, in our case for a default install is: SSH-2.0-OpenSSH_6.6.1p1 Ubuntu-2ubuntu2.10
    listen_endpoints - because we updated our ssh settings so Cowrie can use port 22 for the honeypot we must configure it here, we set tcp:22:interface=0.0.0.0

If running on a cloud instance (GCP,AWS,Azure) modify the following:

    fake_addr = eg. 172.30.xxx.xxx
    internet_facing_ip = eg. 34.217.xxx.xxx

We will leave the output configuration as is for now as we will come back to it in a bit. Save and exit the config and let’s look over at the operating system now.

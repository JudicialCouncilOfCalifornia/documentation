# ======================== local dev env setup ============================
# These are the verified steps to set up a local dev env on Windows 11


# step 0: go to https://aka.ms/wsl2kernel download wsl_update_x64.msi, install it

# step 1: create an instance of Ubuntu-22.04, which is LTS
luanch cmd and run the following
```
wsl --install -d Ubuntu-22.04
wsl --set-version Ubuntu-22.04 2
```


# from now on, all the commands below are executed on linux console


# step 2: do the common parts

sudo apt update 
sudo apt install coreutils
sudo apt -y install software-properties-common 



# step 3: install php
# we are targetting at php 8.1

sudo add-apt-repository ppa:ondrej/php 
# if no response, just press enter

sudo apt –y install php8.1 
# if the above command ran into issue, then consider: sudo apt install --no-install-recommends php8.1

sudo apt install php8.1-cli
# probably the system already has it

sudo apt -y install php7.4 
# 7.4 is still needed by courtyard development
 
sudo update-alternatives --config php
# optional, choose 8.1 as the default one if the current selection is not



# step 4  ------------------ docker install ---------------------

sudo groupadd docker

sudo usermod -aG docker al
# al is my unix unser name, change it to yours

sudo update-alternatives --config iptables
# choose the legacy one

for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
# for pre-installation cleanup


sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings


# configure docker's running env
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# choose Y to confirm


sudo service docker start

sudo service docker status

docker run hello-world
# for verification, expecting "Hello from Docker!"




# step 5  ------------------ git install and repo to local ---------------------

# rsa pair gen
ssh-keygen -b 4096
# set ssh for this wsl u2204 instance on both github.com and patheon


# add this ssh key to github settings | ssh or gpg key
cat ~/.ssh/id_rsa.pub


# pull git@github.com:JudicialCouncilOfCalifornia/trialcourt.git to somewhere local

# for dockerd auto start
echo "sudo service docker start" >> ~/.profile 
echo "cd ~" >> ~/.profile 

# for git installation
sudo add-apt-repository ppa:git-core/ppa 
sudo apt update 
sudo apt install git 



# step 6 -----------------for lando and terminus ---------------------
wget https://files.devwithlando.io/lando-stable.deb
sudo dpkg -i lando-stable.deb

sudo apt-get install php8.1-xml
# please note: use 8.1, otherwise, terminus won't be installed
# type Y to confirm

sudo apt install php8.1-cli
# type Y to confirm

sudo apt-get update
sudo apt-get install php8.1-xml


rm -rf ~/terminus
mkdir -p ~/terminus && cd ~/terminus
curl -L https://github.com/pantheon-systems/terminus/releases/download/3.5.0/terminus.phar --output terminus
chmod +x terminus
 ./terminus self:update

terminus --version
# expecting to see "Terminus 3.5.1"




# step 7 -----------------ssh config ---------------------

touch ~/.ssh/config
vim ~/.ssh/config

# add this content to ~/.ssh/config
Host *.drush.in 
    # The settings on the next two lines are temporary until Pantheon updates the available key types. 
    # If 'PubkeyAcceptedAlgorithms' causes an error, remove it. 
    HostkeyAlgorithms +ssh-rsa 
    PubkeyAcceptedAlgorithms +ssh-rsa 

cat ~/.ssh/config




# step 8 -----------------npm and composer, and lando upgrade ---------------------


sudo apt update 
sudo apt install 
 
wget php-cli php-zip unzip 
wget -O composer-setup.php https://getcomposer.org/installer 

sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer 
sudo apt-get install php7.4-mbstring 

sudo curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash 
source ~/.bashrc 

nvm install 12
nvm install 18

# as of now, if we do "lando version", and the printed version will be 3.1.4, which is too old

# download this from the Alex's shared folder, and put it at your local folder

sudo dpkg -i /mnt/c/Users/Awu/shared/debs/lando-x64-v3.6.4.deb
lando version
# expecting to see 3.6.4




# step 9 -----------------run drupal ---------------------

<!-- go to the git repo, typicall trialcourt, you want to work on -->

# with vpn on, cd to the repo folder
lando composer install
lando rebuild

lando composer global require drush
lando rebuild

lando drush cache-clear drush
# type 0

lando composer install
lando rebuild
lando start


# go to https://dashboard.pantheon.io/personal-settings/ssh-keys to set up the ssh access


lando dbget @inyo.live
# a crash will be seen if pantheon.io's setup is old or not set
# expected to see a db file downloaded to ./data folder

ls -la data/*
# expect to see the file size


lando dbim data/inyo.live-2024-06-25.sql.gz -d inyo
# set the db up, this is only doable after we have a non-empty db file under data


# choose inyo as the sandbox site
lando reset -l @local.inyo.site


lando mysql  
# to verify if inyo databases exists, expecting to see MariaDB [(none)]>
# you may further play with mysql
# use inyo;
# select * from users;
# type 'exit' to exit from mysql console


use the following command to evaluate if db has been started or not
lando logs -s database

if yes then install mysql client and use it to talk to db server
sudo apt update
sudo apt install mysql-client
mysql --host=127.0.0.1 --port=32790 --user=drupal8 --database=inyo --password=drupal8 --silent --execute "select * from users"
# port number can be found using lando info




# gen the url
lando drush @local.inyo cr
lando drush @local.inyo uli

# launch Edge and past the url on the address


<!-- # grant more memory to wls
C:\Users\AWu>type .wslconfig
[wsl2]
memory=8GB -->



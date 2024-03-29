# Easy Install DOMJudge VM
Quick and Easy install DOMJudge with 2 VM (Separated DOMServer and Judgehost)

DOMJudge Official : [DOMJudge](https://www.domjudge.org)
DOMJudge Download : [DOMJudge Download](https://www.domjudge.org/download)

## Install DOMServer VM
Reference : [DOMServer Manual](https://www.domjudge.org/docs/manual/7.3/install-domserver.html)

### 0. Update and Upgrade Repository
```
sudo apt update
```

then

```
sudo apt upgrade
```

### 1. Initialize
```
cd ~
```

### 2. Download DOMJudge
```
wget https://www.domjudge.org/releases/domjudge-7.3.3.tar.gz
tar -xf domjudge-7.3.3.tar.gz
mv domjudge-7.3.3 domjudge
```

### 3. Install Requirements
```
sudo apt install acl zip unzip mariadb-server apache2 \
      php php-fpm php-gd php-cli php-intl php-mbstring php-mysql \
      php-curl php-json php-xml php-zip composer ntp \
      libcgroup-dev libjsoncpp-dev make gcc g++ libcurl4-openssl-dev net-tools
```

### 4. Make DOMServer
```
cd ~/domjudge
./configure --prefix=$HOME/domjudge
make domserver
sudo make install-domserver
```

### 5. Generate Database
```
cd ~/domjudge/domserver/bin
./dj_setup_database genpass
sudo ./dj_setup_database -u root -r install
```

### 6. Setting up Apache (php 7.4)
```
sudo ln -s ~/domjudge/domserver/etc/apache.conf /etc/apache2/conf-available/domjudge.conf
sudo ln -s ~/domjudge/domserver/etc/domjudge-fpm.conf /etc/php/7.4/fpm/pool.d/domjudge.conf
sudo a2enmod proxy_fcgi setenvif rewrite
sudo a2enconf php7.4-fpm domjudge
sudo service php7.4-fpm reload
sudo service apache2 reload
```

### 7. REST API Secret
This is DOMServer Secret Key, will be used in judgehost VM
```
cat ~/domjudge/domserver/etc/restapi.secret
```

### 8. DOMServer Addresss
```
ifconfig
```

## Install Judgehost VM
Reference : [Judgehost Manual](https://www.domjudge.org/docs/manual/7.3/install-judgehost.html)

### 0. Update and Upgrade Repository
```
sudo apt update
```

then

```
sudo apt upgrade
```

### 1. Initialize
```
cd ~
```

### 2. Download DOMJudge
```
wget https://www.domjudge.org/releases/domjudge-7.3.3.tar.gz
tar -xf domjudge-7.3.3.tar.gz
mv domjudge-7.3.3 domjudge
```

### 3. Install Requirements
```
sudo apt install make sudo debootstrap libcgroup-dev lsof \
      php-cli php-curl php-json php-xml php-zip procps \
      gcc g++ default-jre-headless default-jdk-headless \
      ghc fp-compiler \
      libcgroup-dev libjsoncpp-dev make gcc g++ libcurl4-openssl-dev net-tools
```

### 4. Configure Judgehost
```
cd ~/domjudge
./configure --prefix=$HOME/domjudge
make judgehost
```

### 5. REST API Credentials
This need to be done in the **Domserver** not in **Judgehost**. In this step we'll get Domserver restapi.secret information.

```
cat ~/domjudge/judgehost/etc/restapi.secret
```

With above command running in the Domserver, you'll get something like this. Copy the line that have a letter default (in this case line 3).

```
# Randomly generated on host gcloud-3-zydhan, Sat Aug  7 10:29:32 UTC 2021
# Format: '<ID> <API url> <user> <password>'
default http://localhost/domjudge/api   judgehost       XXXXXXXXXXX
```

Copy that line and back to the **Judgehost**. In Judgehost visit `~/domjudge/judgehost/etc/restapi.secret` if it doesn't exist create that file first. After that just copy the line you got from previous Domserver restapi.secret. Change the url into your Domserver API URL.

```
default https://xxx.xxx.xxx.xxx/some/domjudge/url/api   judgehost       XXXXXXXXXXX
```

Tip : Try to go to Domserver IP and test API url connection. If it does return something like this, then it's a valid url.

```
{"api_version":4,"domjudge_version":"7.3.3","environment":"dev","doc_url":"https://xxx.xxx.xxx.xxx/some/domjudge/url/api/doc"}
```

### 6. Make Judgehost
```
sudo make install-judgehost
```

### 7. Add User
```
sudo useradd -d /nonexistent -U -M -s /bin/false domjudge-run
sudo useradd -d /nonexistent -U -M -s /bin/false domjudge-run-0
sudo useradd -d /nonexistent -U -M -s /bin/false domjudge-run-1
```

### 8. Sudo Permissions
```
cd ~/domjudge/judgehost/etc
sudo cp sudoers-domjudge /etc/sudoers.d/
```

### 9. Create chroot environment
```
cd ~/domjudge/judgehost/bin
sudo ./dj_make_chroot
```

### 10. Edit grub
Change line of  ```/etc/default/grub```
```
sudo nano /etc/default/grub
```

Comment ```GRUB_CMDLINE_LINUX_DEFAULT="<something>"``` 
And add this line below previous commented line ```GRUB_CMDLINE_LINUX_DEFAULT="quiet cgroup_enable=memory swapaccount=1"```

Then 
```
sudo update-grub
```

### 11. Reboot
```
sudo reboot
```

### 12. create necessary users
```
cd ~/domjudge/lib/systemd/system
sudo cp domjudge-judgehost.service domjudge-judgehost-0.service
sudo cp domjudge-judgehost.service domjudge-judgehost-1.service

sudo ln -s -f ~/domjudge/lib/systemd/system/* /etc/systemd/system/

sudo sed -i 's/-n 0/-n 0/' domjudge-judgehost-0.service
sudo sed -i 's/-n 0/-n 1/' domjudge-judgehost-1.service
```

### 13. create cgroup
```
cd ~/domjudge/judge
sudo ./create_cgroups
sudo systemctl enable create-cgroups --now
```

### 14. Start judgedaemon
```
sudo systemctl enable domjudge-judgehost-0
sudo systemctl enable domjudge-judgehost-1
sudo systemctl start  domjudge-judgehost-0
sudo systemctl start  domjudge-judgehost-1

sudo systemctl status domjudge-judgehost*
```

## Extra
### DOMServer Address
```
ifconfig
```

### DOMJudge Admin Login
- Username : admin
- Password get from DOMServer VM :
```
cat ~/domjudge/domserver/etc/initial_admin_password.secret
```

### DOMJudge Dummy Team Login
- Username : dummy
- Password : dummy

# References
- https://github.com/DOMjudge/domjudge/issues/855

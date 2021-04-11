# Easy Install DOMJudge VM
Quick and Easy install DOMJudge with 2 VM (Separated DOMServer and Judgehost)
DOMJudge : [DOMJudge](https://www.domjudge.org)
Download : [DOMJudge Download](https://www.domjudge.org/download)

## Install DOMServer VM
Reference : [DOMServer Manual](https://www.domjudge.org/docs/manual/7.3/install-domserver.html)

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
      libcgroup-dev libjsoncpp-dev make gcc g++ libcurl4-openssl-dev
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

## Install Judgehost VM
Reference : [Judgehost Manual](https://www.domjudge.org/docs/manual/7.3/install-judgehost.html)

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
      libcgroup-dev libjsoncpp-dev make gcc g++ libcurl4-openssl-dev
```

### 4. Make Judgehost
```
cd ~/domjudge
./configure --prefix=$HOME/domjudge
make judgehost
sudo make install-judgehost
```

### 5. Add User
```
sudo useradd -d /nonexistent -U -M -s /bin/false domjudge-run-0
sudo useradd -d /nonexistent -U -M -s /bin/false domjudge-run-1
```

### 6. Sudo Permissions
```
cd ~/domjudge/judgehost/etc
sudo cp sudoers-domjudge /etc/sudoers.d/
```

### 7. Create chroot environment
```
cd judgehost/bin
sudo ./dj_make_chroot
```

### 8. Edit grub
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

### 9. Reboot
```
reboot
```

### 10. create cgroup
```
cd ~/domjudge/judge
sudo ./create_cgroups
sudo systemctl enable create-cgroups --now
```

### 11. REST API Credentials
```


```

### 12. Start judgedaemon
```
sudo systemctl enable domjudge-judgehost-0
sudo systemctl start  domjudge-judgehost-1

```
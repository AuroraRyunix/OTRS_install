preconfigued OVFs, for vmware workstation 16 and above, are available in this repo:
[LINK](https://drive.google.com/drive/folders/1sLbEr0CBfe1cEFFms-ZOwzgwxOUsCGBT)

**Manual configuration**
1) make sure debian/ubuntu is up to date
```bash
sudo apt-get update && sudo apt-get upgrade
```
2. install required packages:
```bash
sudo net-tools apt-get install apache2 libapache2-mod-perl2 mariadb-server libdatetime-perl libcrypt-eksblowfish-perl libcrypt-ssleay-perl libgd-graph-perl libapache-dbi-perl libsoap-lite-perl libarchive-zip-perl libgd-text-perl libnet-dns-perl libpdf-api2-perl libauthen-ntlm-perl libdbd-odbc-perl libjson-xs-perl libyaml-libyaml-perl libxml-libxml-perl libencode-hanextra-perl libxml-libxslt-perl libpdf-api2-simple-perl libmail-imapclient-perl libtemplate-perl libtext-csv-xs-perl libdbd-pg-perl libapache2-mod-perl2 libtemplate-perl libnet-dns-perl libnet-ldap-perl libio-socket-ssl-perl libmoo-perl libdbd-mysql-perl -y
```
3. if using ubuntu, remove the default "open-vm-tools" and replace it with "open-vm-tools-desktop"
```bash
sudo apt-get autoremove open-vm-tools
```
4. reboot your system
```bash
sudo apt-get install open-vm-tools-desktop
```
5. reboot your system
copy paste into vmware workstation should work now
edit the mariadb.conf.d/50-otrs.cnf config file using nano, or your prefered editor (gedit is GUI based, hence easier to use, replace "nano" with gedit)
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-otrs.cnf
```
OR
```bash
sudo gedit /etc/mysql/mariadb.conf.d/50-otrs.cnf
```
6. paste the following code into the above mentioned file:
```bash
[mysqld]
max_allowed_packet=64M
query_cache_size=36M
innodb_log_file_size=256M
```
7. restart MariaDB
```bash
sudo service mariadb restart
```
8. create a new root user for mariaDB
```bash
sudo mysql -u root -p
```
9. now you enter the mMarinaDB shell automatically (if not, rerun the above command)
execute the following command in the MarinaDB shell, the shell will quit on it's own ("quit;")
```bash
CREATE DATABASE otrs DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci; 
GRANT ALL PRIVILEGES ON otrs.* TO otrs@localhost IDENTIFIED BY 'kamisama123';
quit;
```
10. enable the required apache2 modules (back in the main linux terminal)
```bash
sudo a2enmod perl
sudo a2enmod headers
sudo a2enmod deflate
sudo a2enmod filter
```
11. restart apache2
```bash
sudo service apache2 restart
```
12. create the OTRS user, and give the required permissions
```bash
sudo useradd -d /opt/otrs -c 'OTRS user' otrs && sudo usermod -aG www-data otrs
```
13. download otrs community edition (v6)
```bash
wget https://otrscommunityedition.com/download/otrs-community-edition-6.0.32.tar.bz2
```
14. extract and move it to the final directory
```bash
tar -jxvf otrs-community-edition-6.0.32.tar.bz2 && sudo mv otrs-community-edition-6.0.32 /opt/otrs
```
15. check if perl installed all the required libs
```bash
sudo perl /opt/otrs/bin/otrs.CheckModules.pl
```
16. create a new OTSR config file
```bash
sudo cp /opt/otrs/Kernel/Config.pm.dist /opt/otrs/Kernel/Config.pm
```
```bash
sudo nano /opt/otrs/Kernel/Config.pm
```
17. fill in the value DatabasePw='PWD' (where PWD is your password)
save the file using ctrl+O, ctrl+X
edit the OTRS config script:
```bash
sudo nano /opt/otrs/scripts/apache2-perl-startup.pl
```
remove the "#' before "use DBD::mysql ();  and "use Kernel::System::DB::mysql;"
so that:
```bash
#use DBD::mysql ();
#Kernel::System::DB::mysql;
```
becomes:
```bash
use DBD::mysql ();
Kernel::System::DB::mysql;
```
18. set the required folder permisions for the otrs directory:
```bash
sudo /opt/otrs/bin/otrs.SetPermissions.pl --web-group=www-data
```
19. enable apache2
```bash
sudo ln -s /opt/otrs/scripts/apache2-httpd.include.conf /etc/apache2/sites-enabled/otrs.conf
```
20. check if every module is correctly loaded and installed:
```bash
sudo perl -cw /opt/otrs/bin/cgi-bin/index.pl 
sudo perl -cw /opt/otrs/bin/cgi-bin/customer.pl
sudo perl -cw /opt/otrs/bin/otrs.Console.pl
```
21. finally, restart apache2
```bash
sudo perl -cw /opt/otrs/bin/cgi-bin/index.pl
sudo perl -cw /opt/otrs/bin/cgi-bin/customer.pl
sudo perl -cw /opt/otrs/bin/otrs.Console.pl
```
(all 3 of hte above should return "syntax OK")
22. open the OTRS installer ON THE MACHINE, using a browser:
```html5
http://localhost/otrs/installer.pl
```
23. continue from slide P37 onwards.

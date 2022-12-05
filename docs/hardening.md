# Hardening machine

## Bastion Host
To secure our servers, we installed some safeguards to prevent unauthorised access. A bastion host is deployed on the public net to mimimise the risk of allowing unauthorised access into the private network

![Diagram](/images/bastion.png)


## Hardening software

- Delete users that are not in use
- Fail2ban
- psacct
- Disable login to account with empty address
- Logwatch
- Lynis
- Disable uncommon filesystems, bluetooth,appletalk,Ipv6 and uncommon protocols
- Enable firewall and open ports for wireguard
- Remove ssh root login

```
# 6. Server hardening

# Delete users that are not needed
sudo userdel -r mail
sudo userdel -r news
sudo userdel -r games
sudo userdel -r lp

#Install and start Fail2ban
sudo apt install -y fail2ban
sudo systemctl enable fail2ban.service
sudo systemctl start fail2ban.service

#Install psacct
sudo apt-get install acct
sudo /etc/init.d/acct start

#Diable ATD and ABRTD services (Automatic Bug Reporting Tool)
sudo systemctl disable atd.service
sudo systemctl disable abrtd.service

#Setup Warning Banner for SSH Access
sudo echo "Unauthorized access prohibited. Logs are recorded and monitored" > /etc/issue
sudo echo "Unauthorized access prohibited. Logs are recorded and moniored" > /etc/issue.net

#Prevent login to accounts with empty passwords
sudo sed --follow-symlinks -i 's/\<nullok\>//g' /etc/pam.d/system-auth
sudo sed --follow-symlinks -i 's/\<nullok\>//g' /etc/pam.d/password-auth

#Install Logwatch
sudo DEBIAN_FRONTEND=noninteractive apt-get -yq install logwatch

#Install Lynis (rootkit scanner and security auditing tool)
sudo apt install -y lynis

#Disable uncommon filesystems, bluetooth,appletalk,Ipv6 and uncommon protocols
(
cat <<-EOF
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install squashfs /bin/true
install udf /bin/true
install fat /bin/true
install vfat /bin/true
install cifs /bin/true
install nfs /bin/true
install nfsv3 /bin/true
install nfsv4 /bin/true
install gfs2 /bin/true
install bnep /bin/true
install bluetooth /bin/true
install btusb /bin/true
install net-pf-31 /bin/true
install appletalk /bin/true
options ipv6 disable=1
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true
EOF
) | sudo tee /etc/modprobe.d/hardening.conf

# Enable firewall and open ports
sudo ufw --force enable
sudo ufw default deny incoming
sudo ufw allow xxxx
sudo ufw allow xxxx
sudo ufw allow xxxx
sudo ufw allow xxxx:xxxx/tcp

# SSH Config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.BAK

echo "Port xxxx
PermitRootLogin no
PermitEmptyPasswords no
X11Forwarding no
LoginGraceTime 2m
MaxAuthTries 3
PasswordAuthentication no
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no" | sudo tee /etc/ssh/sshd_config
sudo service sshd restart
```
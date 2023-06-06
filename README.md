# Samba that appear on window explorer

Samba server on linux enviroment provide the windows like file sharing on linux machine which used cifs/smb protocols to provide easiest way to share files and folder with almost every Major OS.

# Why

##### There are many forms and shares available that cover the Samba share better then this repo but they did't appear in window explorer network section so i just cover it in a simplest way. 
------------

# Tested Distros

| S. No. | Distro | Version | Status | Package Manager | 
| :-----: | ------ | ------- | ------ | :--------: |
| 1. | Centos Linux | 7/8/9 | [x] | RPM |
| 2. |  Ubuntu Linux |  18.04.6 LTS/20.04.4 LTS/22.04 LTS/ 23.04 | [x]  | DEB |
| 3. | Rocky Linux | 8/9 | [x] | RPM |
| 4. | Oracle Linux | 7/8/9 | [x] | RPM |
| 5. | Debian Linux | 10/11 | [x] | DEB |

## Installing samba on machine

#### On rpm base distro 

* wssd help your samba server to  appear in Windows Explore Network section again.

```bash
sudo yum install epel-release -y
sudo yum install samba wsdd -y 
```

-  on deb base distro
```shell
sudo apt install samba wsdd -y
```

#### Starting smb service and enable it to autostart
```shell
sudo systemctl enable smb nmb wsdd
sudo systemctl start smb nmb wsdd
```


#### Configuring Firewall

-  **on deb base distro you dont need to config firewall**
```shell
sudo firewall-cmd --add-service=samba --zone=public --permanent
sudo firewall-cmd --add-service=wsdd --zone=public --permanent
sudo firewall-cmd --reload 
```


#### Config file edit for the public share

*copy and paste all the command in linux terminal you can change you path = YOUR-media-location*

```shell
echo "
[global]
    workgroup = WORKGROUP
    lanman auth = yes
    client lanman auth = yes
    client plaintext auth = yes
    client min protocol = nt1
    client max protocol = smb3
    log file = /var/log/samba/log.%m
    max log size = 1000
    panic action = /usr/share/samba/panic-action %d
    map to guest = bad user

[My Public Share]
    comment = My Public Folder
    path = /media
    public = yes
    writable = yes
    create mast = 0755
    guest ok = yes
    security = SHARE
     " | sudo tee /etc/samba/smb.conf
```

#### Config file edit for the private share with username/password

- only use this section when you want to use your share with user and password else dont use it.

```shell
echo "
[global]
    workgroup = WORKGROUP
    lanman auth = yes
    client lanman auth = yes
    client plaintext auth = yes
    client min protocol = nt1
    client max protocol = smb3
    log file = /var/log/samba/log.%m
    max log size = 1000
    panic action = /usr/share/samba/panic-action %d
    map to guest = bad user
		 
[homes]
    comment = Home Directories
    valid users = %S, %D%w%S
    browseable = No
    read only = No
    inherit acls = Yes
     " | sudo tee /etc/samba/smb.conf
```

## adding new user and set password

- replace USERNAME with your user name.

```shell
sudo useradd UserName
sudo usermod -s /sbin/nologin UserName
sudo smbpasswd -an UserName
```

## Permissions 

#### [SELinux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux) Configuration

* you don't need to follow this section if you disabled SELinux on you machine

```bash
sudo yum install policycoreutils-python-utils.noarch -y
sudo chcon -Rt samba_share_t /media
sudo smbpasswd -an nobody
sudo chown -R nobody:nobody /media
```
- if you use your share with username and password 

``` shell
sudo setsebool -P samba_export_all_rw=1
sudo setsebool -P samba_enable_home_dirs 1
```


#### Set files and permissions 

```bash
sudo chown -R nobody:nobody /media
sudo chmod -R 775 /media

```

#### Restart the Services

```bash
sudo systemctl restart smb nmb wsdd
```

#### Set your Machine name

```shell
sudo hostnamectl set-hostname My-Nas
```

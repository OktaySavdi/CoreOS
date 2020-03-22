
## Run/Install Fedora CoreOS (FCOS) on VmWare

Fedora Core image is downloaded. [CoreOS](https://getfedora.org/coreos/download/)

Mount the iso file to the server and run the machine.
![image](https://user-images.githubusercontent.com/3519706/77246525-50a2be80-6c39-11ea-85cb-ed5a3d57fd9d.png)

When you see Fedora CoreOS (Live) screen press the `TAB` button and fill the section ip with your own information

    ip=10.10.10.10::10.10.10.254:255.255.255.0:master.local:eth0:none nameserver=10.10.10.01

![image](https://user-images.githubusercontent.com/3519706/77046450-12f12c00-69d4-11ea-9ff7-412336b40224.png)

the installation step will start

![image](https://user-images.githubusercontent.com/3519706/77046594-53e94080-69d4-11ea-9356-63fd24341269.png)

Necessary checks are made, 

- The machine receives IP, 
- The machine can access the internet

The machine receives IP, 
![image](https://user-images.githubusercontent.com/3519706/77246597-12f26580-6c3a-11ea-8c92-1e06fd2cfcc1.png)

It is checked whether there is internet access with the command `curl www.google.com`.
![image](https://user-images.githubusercontent.com/3519706/77056521-9108ff00-69e3-11ea-8296-483bc1b808f4.png)

If the machine cannot access the internet (dns address) or not received a ip, method, dns and gateway assignments can be made with the following commands.

```json
nmcli connection                                    # check connection
nmcli con mod "Wired Connection 1" iframe eth0      #change iframe name
nmcli con mod "Wired Connection 1" con-name eth0    #change connection name if not eth0  
nmcli con mod eth0 ipv4.addresses 10.10.10.10/24    #add ip addres   
nmcli con mod eth0 ipv4.gateway 10.10.10.254        #add gateway addres    
nmcli con mod eth0 ipv4.method manual               #add Method addres  
nmcli con mod eth0 ipv4.dns 10.10.10.01             #add dns addres    
nmcli con down eth0
nmcli con up eth0
```

Password is set and hash information is obtained for the core user
```bash
sudo openssl passwd -1 > cloud-config-file # password_hash is added in config.fcc file
```
Password is entered.

Password hash information is obtained with the command `cat cloud-config-file`.
![image](https://user-images.githubusercontent.com/3519706/77246641-6c5a9480-6c3a-11ea-97fd-d74a7363e3fc.png)

Public key is created for `ssh_authorized_keys` and saved to a safe location (I used 'PuTTYgen')
![image](https://user-images.githubusercontent.com/3519706/77048325-48e3df80-69d7-11ea-8b4c-8dcf7e416e49.png)

The file `config.fcc` is created
```yaml
variant: fcos    
version: 1.0.0    
passwd: 
  users:
  - name: core
    password_hash: $1$Esi2Tm/F$bjR2dJ6gU1tl/qZHIIEuz. 
    ssh_authorized_keys:
    - ssh-rsa AAAAB3Nz<Public Key>L0KNeJXLmw== core
```

The file `config.fcc` is converted to` config.ign` by the following command.
```bash
podman run -i --rm quay.io/coreos/fcct:release --pretty --strict < config.fcc > config.ign
```

The file is displayed with the `cat config.ign` command and its contents are copied.

The file format should be as follows.
```json
{
  "ignition": {
    "config": {
      "replace": {
        "source": null,
        "verification": {}
      }
    },
    "security": {
      "tls": {}
    },
    "timeouts": {},
    "version": "3.0.0"
  },
  "passwd": {
    "users": [
      {
        "name": "core",
        "passwordHash": "$1$Esi2Tm/F$bjR2dJ6gU1tl/qZHIIEuz.",
        "sshAuthorizedKeys": [
          "ssh-rsa AAAAB3Nz<Public Key>L0KNeJXLmw== core"
        ]
      }
    ]
  },
  "storage": {},
  "systemd": {}
}
```
The `config.ign` file is created on the server with the same content.

`Config.ign` is written to disk with the following command
```bash
sudo coreos-installer install /dev/sda -i config.ign
```
The disc is then ejected and the server is restarted.

wait for dhcp discover for timeout

![image](https://user-images.githubusercontent.com/3519706/77055189-864d6a80-69e1-11ea-85c7-ab8e235dde1a.png)

The operating system will be turned on in a few minutes.

You can login with the user you have defined.

![image](https://user-images.githubusercontent.com/3519706/77246483-bb9fc580-6c38-11ea-86dc-d59294c61942.png)

Internet access is checked again with `curl www.google.com`

If there is no internet access, the commands issued are run again by editing the ip.

```json
nmcli connection                                    # check connection
nmcli con mod "Wired Connection 1" iframe eth0      #change iframe name
nmcli con mod "Wired Connection 1" con-name eth0    #change connection name if not eth0  
nmcli con mod eth0 ipv4.addresses 10.10.10.10/24    #add ip addres   
nmcli con mod eth0 ipv4.gateway 10.10.10.254        #add gateway addres    
nmcli con mod eth0 ipv4.method manual               #add Method addres  
nmcli con mod eth0 ipv4.dns 10.10.10.01             #add dns addres    
nmcli con down eth0
nmcli con up eth0
```
To connect CoreOS as a remote to putty

Connection> SSH> Auth Private Key is added

![image](https://user-images.githubusercontent.com/3519706/77055456-f78d1d80-69e1-11ea-9b25-ac643784ccc8.png)

![image](https://user-images.githubusercontent.com/3519706/77055516-155a8280-69e2-11ea-811c-e758c776222f.png)

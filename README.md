# Call Of Duty 4 - Dedicated Linux Server CentOS 7 SELinux Policy Module

This is a policy module built on CentOS 7.2 64-bit. Using Reference policy. For systemd.

Since the dedicated Linux daemon is from 2005 and without any recent patches. It's a good idea to use SELinux to lock it down as best we can.

Obviously, this is all provided in good will. I cannot be held responsible. It's your job to make sure you read over everything and not blindly install stuff from the internet.

If you have any additions, feel free to add a pull request.

Other Call Of Duty Module's also available. See my github.

## Features and Assumptions
* This module was made on a CentOS 7.2 64-bit Server. **No assumptions** were given to be completely portable to other Operating Systems.
* You have an understanding how SELinux works.
* All server files are kept in the home directory we create.
* Directory names are exactly as the ones listed in the file context. ".fc".
* Since systemd will be starting the cod4 server, you won't have a local console. If you need to issue rcon commands, you'll need to join the game.
* Logs will be in the ~/.callofduty4 directory by default. Not within the cod4-srv dir.
* Server runs on port 28962 udp.
* Completely systemd operated, start and stop with automatic restart on failure.
* You need to modify the IP in the service file


## Installation
```sh
# Clone the repo
git clone https://github.com/georou/cod4-selinux.git

# Create a user account with no login shell
useradd -s /sbin/nologin cod4

# Create a directory and copy in your server files to it
mkdir -v /home/cod4/cod4-srv

# Upload the .service file
cp -v cod4-srv.service /etc/systemd/system
systemctl daemon-reload

# Install the SELinux policy module. Compile it before hand to ensure proper compatibility (see below)
semodule -i cod4.pp

# Restore all the correct context labels
restorecon -v /etc/systemd/system/cod4-srv.service
restorecon -Rv /home/cod4

# Add the port to SELinux (the port can be different, also change it in the service file)
semanage port -a -t cod4_port_t -p udp 28962

# Open require firewall ports
firewall-cmd --permanent --add-port=28961/udp
OR
iptables -I INPUT -m conntrack --ctstate NEW,ESTABLISHED -p udp --dport 28962 -j ACCEPT

# Start the server
systemctl enable cod4-srv.service
systemctl start cod4-srv.service
```

## How To Compile The Module Locally(Recommended before installing)
Ensure you have the `selinux-policy-devel` package installed.
```sh
# Ensure you have the devel packages
yum install selinux-policy-devel
# Change to the directory containing the .fc & .te files
cd cod4-selinux
make -f /usr/share/selinux/devel/Makefile cod4.pp
```

## Optional Security Steps

To further secure the user account of cod4, running as a confined user would be beneficial. user_u is very confined, allowing only the basic system access to work.
```sh
semanage login -a -s user_u cod4
```

## Debugging and Troubleshooting

* If you're getting permission errors, uncomment permissive in the .te file and try again. Re-check logs for any issues.

* Easy way to add in allow rules is the below command, then copy or redirect into the .te module. Rebuild and re-install:
* Don't forget to actually look at what is suggested. audit2allow will most likely go for a coarse grained permission!
* Check man audit2allow for different switches. Either use ref policy or not.
```sh
ausearch -m avc -ts recent | audit2allow -R
# If you get a could not open interface info [/var/lib/sepolgen/interface_info] error, install:
yum install policycoreutils-devel
```

## Future To Do

* Allow external access to the log file - Will need a new cod4_log_t type and interface.

## Compatibility Notes
Built on CentOS 7.2 at the time with:
```
policycoreutils-python-2.5-11.el7_3.x86_64
selinux-policy-3.13.1-102.el7_3.13.noarch
selinux-policy-targeted-3.13.1-102.el7_3.13.noarch
policycoreutils-2.5-11.el7_3.x86_64
selinux-policy-devel-3.13.1-102.el7_3.13.noarch
```

# Setup files for Assetto Corsa Competizone Oracle server

> [!IMPORTANT]
> Please make sure to run `iptables -I INPUT -j ACCEPT` on the instance first to allow all traffic in.
> If you are comforable using `sudo iptables -S` and checking properly, then please make more advanced rules.
> See https://www.reddit.com/r/oraclecloud/comments/q2iv2h/eli5_how_to_forward_ports_on_oracle_cloud/ for more

Scripts to load server setup as follows:
(be prepared for Steam Guard to prompt you the first time you log in on a given instance.)

0. Make sure your Ingress Rules are setup on your Oracle Cloud Instance. 

> [!CAUTION]
> The **source** part of your rule must be *blank* or set to *ALL* to make sure that this works. 
> This is imperative as there is no NAT on your instance so this is the only way to ensure traffic is allowed in.

| Stateless | Source | Protocol | Source | Destination | Notes |
| --- | --- | --- | --- | --- | --- |
| No | 0.0.0.0/0 | TCP | All | 9600 | |
| No | 0.0.0.0/0 | UDP | All | 9600 | |
| No | 0.0.0.0/0 | UDP | All | 9601 | |
| No | 0.0.0.0/0 | TCP | All | 8081 | (believed optional - added for safety) |
| No | 0.0.0.0/0 | UDP | All | 8081 | (believed optional - added for safety) |

1. `update_server.sh`

```bash
#!/bin/bash

steamcmd \
+@sSteamCmdForcePlatformType windows \
+force_install_dir /home/ubuntu/acc \
+login CHANGE_THIS_TO_YOUR_STEAM_ACCOUNT CHANGE_THIS_TO_YOUR_PASSWORD \
+app_update 1430110 \
+quit
```

2. `run_server.sh`

```bash
#!/bin/bash

# Clone configs
rm -rf servers-acc_config/
git clone https://github.com/rgbalex/servers-acc_config.git

# Copy configs to server
rm -rf cfg/
mkdir cfg
cp servers-acc_config/*.json cfg/

# Edit the admin password in settings.json
sed -i '/    "adminPassword": "super_secret_admin_password!",/c\    "adminPassword": "CHANGE_THIS_TO_YOUR_PASSWORD",' cfg/settings.json
sed -i '/    "publicIP": ""/c\    "publicIP": "CHANGE_THIS_TO_YOUR_INSTANCE_PUBLIC_IPV4_ADDRESS"' cfg/configuration.json

# Check the server is up to date
./update_server.sh

# Run the server
wine acc/server/accServer.exe
```

> [!IMPORTANT]
> Don't forget to `chmod +x *.sh` to make these files runnable by default.
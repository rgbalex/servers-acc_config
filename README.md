# Setup files for Assetto Corsa Competizone Oracle server
## To be setup by yourself for free, forever!

Start here for more info: https://developer.valvesoftware.com/wiki/SteamCMD#Linux

> [!NOTE]  
> Sometimes, the all important `RegisterToLobby succeeded` line seems to arrive 30-60 seconds late. Please be patient but do wait for it as without it the server will not be responsive in ACC

> [!TIP]
> ~~Please make sure to run `iptables -I INPUT -j ACCEPT` on the instance first to allow all traffic in.~~
> ~~If you are comforable using `sudo iptables -S` and checking properly, then please make more advanced rules.~~
> 
> This has been added to the startup script and is not important anymore. However, feel free to make your own edits if you deem it necessary.
> See https://www.reddit.com/r/oraclecloud/comments/q2iv2h/eli5_how_to_forward_ports_on_oracle_cloud/ for more
> See https://steamcommunity.com/app/805550/discussions/0/2946998508797770826/ for more

Scripts to load server setup as follows:
(be prepared for Steam Guard to prompt you the first time you log in on a given instance.)

0. Make sure your Ingress Rules are setup on your Oracle Cloud Instance. 

> [!CAUTION]
> The **source** part of your rule must be *blank* or set to *ALL* to make sure that this works. 
> This is imperative as there is no NAT on your instance so this is the only way to ensure traffic is allowed in.

| Stateless | Source    | Protocol | Source | Destination | Notes                                  |
| --------- | --------- | -------- | ------ | ----------- | -------------------------------------- |
| No        | 0.0.0.0/0 | TCP      | All    | 9600        | Must be the same as UDP                |
| No        | 0.0.0.0/0 | UDP      | All    | 9600        | Must be the same as TCP                |
| No        | 0.0.0.0/0 | TCP      | All    | 9601        | Must be one higher than the above port |
| No        | 0.0.0.0/0 | UDP      | All    | 9601        | Must be one higher than the above port |

> [!IMPORTANT]
> If the server does get to the part where it makes a request to the lobby and claims it succeeds, but then silently and immediately crashes with no error, this is because the server is failing to make a handshake to the Assetto Corsa Main Server. The handshake happens on one port higher than the UDP and TCP ports set in the config file.
>
> These ports must be the same because we are using the `publicIP` setting in `settings.json`
>
> See https://www.acc-wiki.info/wiki/Server_Configuration for more information

1. `run_server.sh`

```bash
#!/bin/bash

echo Waiting 10 seconds after boot to start...
sleep 10

# Ensure inbound traffic is allowed
sudo iptables -I INPUT -j ACCEPT

# Since this is called as a service, ensure we are in the right folder
cd /home/ubuntu

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
steamcmd \
+@sSteamCmdForcePlatformType windows \
+force_install_dir /home/ubuntu/acc \
+login CHANGE_THIS_TO_YOUR_STEAM_ACCOUNT CHANGE_THIS_TO_YOUR_PASSWORD \
+app_update 1430110 \
+quit

# Run the server
wine acc/server/accServer.exe
```

> [!IMPORTANT]
> Don't forget to `chmod +x *.sh` to make these files runnable by default.

## To enable autostart

After creating these files in your users home directory (ubuntu by default for Oracle instances), we want to make sure this boots every time this instance is booted. 

After verifying these things work, simply create a service by running the following:

1.  `sudo nano /etc/systemd/system/ACCServer.service`

    Paste the following:

    ```conf
    [Unit]
    Description=ACC Server Service

    [Service]
    ExecStart=/home/ubuntu/run_server.sh

    [Install]
    WantedBy=multi-user.target
    ```

2. Make sure to reload with `sudo systemctl daemon-reload`

3. Start with `sudo systemctl start ACCServer`

4. Check the status with `systemctl status ACCServer`

5. Enable autostart with `sudo systemctl enable ACCServer`


Congratulations, you should now have a free Assetto Corsa Competizone server forever.
Reboot your instance and observe - remember to give it a minute to register the session with the ACC host server. 

# Truenas Scale Initial Setup

After Installing truenas scale electric eel, log in as admin and create a new user by going to the *Credentials* tab on the sidebar and then clicking on *Users*. Next click on the *Add* button on the top right and then add a new user

![image](https://github.com/user-attachments/assets/eb6b96e8-9c65-48d5-9d84-6b8148f954b4)

Set the user name to be similar to the username of your windows pc so that logging into the smb shares will be simple.

![image](https://github.com/user-attachments/assets/270a4283-7622-49d8-bc3d-7c3fcbace6ba)

Next go to the *System* tab on the sidebar and click on *Services* . Enable *SMB* and *SSH* services and turn on the *Enable Automatically* option as well.

![image](https://github.com/user-attachments/assets/87cb5cb3-82de-421e-8b79-a0da2ca72582)

Now, go to the *Data Protection* tab on the sidebar and create **Short** and **Long** Periodic S.M.A.R.T tests. Short tests should run once a week and Long tests should run once a month.

![image](https://github.com/user-attachments/assets/5e03e0dd-c9a2-4819-a39f-5e389c1d18e3)

Next go to the *Storage* tab and create a new pool. give the pool a name and set the zfs raid option as necessary.

![image](https://github.com/user-attachments/assets/cdfb4ea6-9ca0-4db1-8fc0-40636f2c8a62)

Now go to the *DataSets* tab and create a couple of base datasets inside the primary pool. The important ones are **configs** , **media** , **photos** and **torrents** . You can create more datasets in you want but these 4 are mandatory for setting up a media stack and other docker applications. These 4 datasets should be created by using the **Dataset Preset** as **Apps** which you will see when you click on the *Add Dataset* button. By using this preset, we are able to set the built in user **Apps** role permissions to these datasets . This will be useful when we start creating the docker apps. 

![image](https://github.com/user-attachments/assets/190c722f-8474-4e44-a26b-6e3edc2b2b7a)

All the following datasets should be created inside the *configs* datasets. But some of these child datasets need to have permissions set that are different from the rest. The following datasets need to have the **Apps** preset selected when creating them:

- Authelia
- bazarr
- ddns-updater
- firefox
- jellyfin
- lidarr
- portainer
- prowlarr
- qbittorrent
- radarr
- sabnzbd
- sonarr
- speedtest
  - certs
- uptime-kuma
- vaultwarden

The rest of the remaining ones need to be created by selecting the **Generic** dataset preset when creating them. This will give them root permissions and this is necessary for those apps.

- filebrowser
- homepage
- nginx
  - certs

![image](https://github.com/user-attachments/assets/4aa5a921-af70-44a8-b9b5-d60f9c64bc43)

Now we have to add some folders inside the **media** dataset to store our respective media. Go to the *System* tab and click on *Shell*. change over to the root user by typing `sudo su` and then enter your password. Navigate to the pool which will be `/mnt/<pool-name>` and go to the media dataset which will be `/mnt/<pool-name>/media` . You can create folders for your movies and shows as needed. Refer to trash guides to understand how folder structure should be created for different types of media . For referencem, this is how I have created my folders :

- media
  - Animated
    - Animated_Movies
    - Animated_Shows
  - Anime
    - Anime_Movies
    - Anime_Shows 
  - Movies
    - English
    - Regional
  - Music
  - Tv_Shows

After creating the folders that you want, go back up to the media folder and then type these commands

`chown apps media -R`

`chgrp apps media -R`

This will change everything inside the media folder so that the user *apps* that the applications that we set up will use will have access to these folders. 

Now that you have created all the folders and datasets required. Go to the *Shares* tab and create SMB shares for the folders.

![image](https://github.com/user-attachments/assets/03814626-5c87-435e-88c7-4c8abd876f5d)

We can now finally start setting up portainer and install docker apps on it. Go to the *Apps* tab and click on the configuration button on the top right. You should see an option to initialize the apps in a pool. Select the pool that you have created. After this click on *Discover Apps* and search for portainer and proceed to install it. Set the **Portainer Data Storage** to be **Host Path** and set it to be the portainer folder inside the configs folder that you have created. 

![image](https://github.com/user-attachments/assets/67937822-fec4-4c93-8f62-0726f8099092)

Open the portainer app and create a username and password. Then click on *Get Started* and then click on the *Local* environment. Click on the *Stacks* tab and then click on *Add Stack* to add the necessary stacks using the docker compose files. 

Modify the folder paths by adding your `<pool name>` and add the relevent api keys.

![image](https://github.com/user-attachments/assets/77154e41-9774-484d-9304-e28e7e2f0121)

![image](https://github.com/user-attachments/assets/7c076c9a-210b-4aa2-a06a-ac3e8d66f3e2)

![image](https://github.com/user-attachments/assets/8e6ab667-4d77-4659-b8b2-f928bbb5bb95)

Before installing Nginx Proxy go the *System* tab and click on *General Settings*. Change the **Web Interface HTTP Port** from 80 to something else (eg: 91) and change the **Web Interface HTTPS Port** from 443 to something else (eg: 444). Also change the time zone to youre preference. This will make sure that you will still be able to access your truenas dashboard but now you have have to go to `<ip-address>:444`


# App Specific Settings

- For DDNS-Updater - Paste the following into the **config.json** file :
```
  {
  "settings": [
    {
      "provider": "cloudflare",
      "zone_identifier": "20ebed1f08d517b878dc7d51fcaa7a50",
      "domain": "*.m3tal.xyz",
      "ttl": 60,
      "token": "Bh2IvRhOSwZF0yvJW8z7W5M6Np_v8Khn6CYM4AMH",
      "ip_version": "ipv4"
    }
  ]
}
```

- For FileBrowser - Before installing the app using the docker-compose file, create these 2 files inside the *filebrowser* folder - **settings.json** and **filebrowser.db** by using the `touch` command. Paste the following code into the **settings.json** file -
```
{
  "port": 80,
  "baseURL": "",
  "address": "",
  "log": "stdout",
  "database": "/database/filebrowser.db",
  "root": "/srv"
}

```

- For Homepage - Adjust the `settings.yaml` ,  `widgets.yaml` and `services.yaml` to customize the Homepage dashboard.

- For Photoprism - After installing the app, go to the container shell by clicking on the console button inside the `photoprism-photoprism-1` container and type this command to reset the admin accout password - `photoprism passwd admin`

- For QBitTorrent - After installing the app,
  - Check the container logs - *either inside portainer or in dozzle* to get the username and password
  - Click on *Tools* in the toolbar and then click on *Options*.
  - Now navigate to the *Downloads* tab and change the **Default Save Path** to `app/qBittorrent/downloads` .
  - Go to the *BitTorrent* tab and then check the **When ratio reaches** box and set the value as `0` .
  - Go to the *WebUI* tab and then change the default **Username** and **Password** . Also Check the following checkboxes - **Bypass authentication for clients on localhost** and **Bypass authentication for clients in whitelisted IP subnets** and enter `192.168.0.0/24` to bypass the login screen on localhost.
 

# Reading Material

 - Refer https://github.com/geekau/mediastack for more details
 - Refer to trash guides - https://trash-guides.info/ for setup and customizing all *arr* apps
 - Refer to https://wiki.serversatho.me/ and https://www.youtube.com/@ServersatHome/videos for truenas setup and apps setup.

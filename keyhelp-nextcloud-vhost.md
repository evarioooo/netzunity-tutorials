<p align="center">
    <a href="https://www.evarioo.de" target="_blank"><img src="https://i.ibb.co/sPVdY46/github-readme-logo.png" width="493" /></a><br>
    <img src="https://img.shields.io/github/stars/evarioooo/keyhelp-nextcloud-vhost?style=for-the-badge&label=Repo%20stars" />
</p>

# Are you having some trouble?

Do you really need some help and are stuck? Don't hesitate and just contact me in Discord or alternatively by email:

[<img src="https://img.shields.io/badge/h4zebust3r90-me?style=for-the-badge&logo=telegram&logoColor=white&labelColor=%23ffba13&color=grey">](https://t.me/h4zebust3r90)<br>
[<img src="https://img.shields.io/badge/h4zebust3r90-me?style=for-the-badge&logo=discord&logoColor=white&labelColor=%23ffba13&color=grey">](https://discord.gg/9qqKZuAbsa)<br>
[<img src="https://img.shields.io/badge/evarioo_x-me?style=for-the-badge&logo=x&labelColor=%23ffba13&color=grey">](https://x.com/evarioo_x)<br>
[<img src="https://img.shields.io/badge/hello%40evarioo.de-me?style=for-the-badge&logo=maildotru&labelColor=%23ffba13&color=grey">](mailto:hello@evarioo.de?subject=Kontakt%20%C3%BCber%20GitHub)

# Nextcloud installation in keyhelp

I had a lot of difficulties with Nextcloud settings in conjunction with Keyhelp. I don't want to make a statement about the two software(s), it's just the reason why you have to read this crap.

## Setup keyhelp domain/user

To begin with, it is enough to create a new domain in Keyhelp. Personally, I recommend having your own user for the Nextcloud instance as you have to make some custom settings.

- Add "Security settings / SSL settings:

Enable SSL support
Set HSTS to 15552000 seconds (or 180 days)

![alt text](https://i.ibb.co/s9cgDW9/IMG-1395.jpg)

- Add "Additional php configuration" with the below settings:
```
opcache.enable_cli=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=1
```

## Setup nextcloud instance

### Install nextcloud
- Download nextcloud web installer (community project)
```
https://download.nextcloud.com/server/installer/setup-nextcloud.php
```

- Use Keyhelp's built-in file manager to upload the "setup-nextcloud.php" to the designated folder with the new user (from step 1).

![alt text](https://i.ibb.co/GWtzMWS/keyhelp-files.png)

> [!TIP]
> To view the file manager you have to log in to Keyhelp as the user who owns the domain :)

> [!TIP]
> You should also give the file “occ” the rights 744.
![alt text](https://i.ibb.co/zNvGVQR/keyhelp-chmod.png)

- Now go to
```
https://YOURDOMAIN.TLD/setup-nextcloud.php
```
in your browser and let the Nextcloud assistant guide you through the rest of the process.

> [!TIP]
> Personally, I advise you to put the "data" directory not in the www but in the files folder

### Setup memcache & redis

- Install APCU for php
```
sudo apt install php-apcu
```

- Install redis server
```
sudo apt install redis-server php-redis
```

- Now open redis configuration
```
sudo nano /etc/redis/redis.conf
```

Change 
```
port 6379
```
to
```
port 0
```
Change
```
# unixsocket /var/run/redis/redis-server.sock
```
to
```
# unixsocket /var/run/redis/redis-server.sock
```
From the line below make
```
unixsocketperm 770
```

We'll stop inside for a moment... remember? So we save and close the file ;)

- Now we're adding your new keyhelp domain user to the redis group
```
sudo usermod -a -G redis <yourKeyhelpDomainUser>
```

- Restart your redis & apache2 server
```
sudo service redis-server restart
sudo service apache2 restart
```

- Last but not least edit your nextcloud config.php stored in config directory of your nextcloud installation

```
'memcache.local' => '\\OC\\Memcache\\APCu',
'filelocking.enabled' => 'true',
'memcache.locking' => '\\OC\\Memcache\\Redis',
'redis' => array (
    'host' => '/var/run/redis/redis-server.sock',
    'port' => 0,
),
```

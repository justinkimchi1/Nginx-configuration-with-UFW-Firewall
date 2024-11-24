# Assignment-3
In this assignment we will be performing tasks such as creating new system users to run `.service` and `.timer` scripts on a Nginx server with a UFW firewall. 

## Task 1 Creating a System User
Here we will create a system user with a specified home directory and a login shell appropriate for a non-login user. 
1. Creating the system user with specified home and shell. [[1]](#1-useradd-command)
```
sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen
```
- `-r`: creates system account
- `-d`: specifies users home directory
- `-s`: specifies users default shell, in this case we use `/usr/sbin/nologin` to prevent users from logging into the account

2. Git clone repository

In your home directory run this command to get the needed files from the repository
```
git clone https://git.sr.ht/~nathan_climbs/2420-as2-start
```

3. Creating the correct directory structure using `-p` option. [[2]](#2-mkdir-command)
```
sudo mkdir -p /var/lib/webgen
```
> This creates `webgen`'s home directory where we will create the other directories 
- `-p`: option creates parent directories as needed

Then create the other directories with this:
```
sudo mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML
```
Now `cd` into the 2420-as2-start starter directory that we cloned in the previous step:
```
cd 2420-as2-start
```

And move the `generate-index` file into your new directory with this:
```
sudo mv generate-index /var/lib/webgen/bin/
```

Now create the index.html file
```
sudo touch /var/lib/webgen/HTML/index.html
```

Finally we will give the user `webgen` ownership of the directory and its files [[3]](#3-chown-command)
```
sudo chown -R webgen:webgen /var/lib/webgen
```
- `-chown`: used to change user ownership of files
- `-R`: option to recursively change ownership of directories and their contents
- `webgen:webgen /var/lib/webgen`: assign webgen and the group webgen to the specified directory 

You have finished creating a system user with the appropriate directory structure and ownership!

## Task 2 Creating a `.service` and `.timer` scripts
1. Create the `generate-index.service` file 
```
sudo nvim /etc/systemd/system/generate-index.service
```
Paste this into the file [[4]](#4-systemdtimers)
```bash
[Unit]
Description="Runs the generate_index script everyday at 5:00"

[Service]
ExecStart=/var/lib/webgen/bin/generate-index
User=webgen
Group=webgen
```
- `[Unit]`: section that provides information about the service. We have the description in there
- `[Service]`: section where we define the specifics about how the service should run. Here we give it the path to the file that we want to run and which user and group we want it to be run by

2. Create the `generate-index.timer` file
```
sudo nvim /etc/systemd/system/generate-index.timer
```

Paste this into the file [[4]](#4-systemdtimers)
```bash
[Unit]
Description="Run the generate-index.service at 5:00 am everyday"

[Timer]
OnCalendar=*-*-* 5:00:00
Unit=generate-index.service

[Install]
WantedBy=timers.target
```
- `[Unit]`: section that provides information about the service. We have the description in there
- `[Timer]`: section that we can define the scheduling of the timer. Here we set it to activate every day at 5:00 am. We also specify which unit to activate, which is our `generate-index.service` file
- `[Install]`: section that defines how the unit should be enabled within the `systemd` system. Here we tell `systemd` to automatically start our timer when it initializes `timers.target`, which is responsible for managing all timer units

3. Activate `generate-index.timer` file [[5]](#5-systemctl)
```
sudo systemctl start generate-index.timer
```
Enable it to boot on start as well
```
sudo systemctl enable generate-index.timer
```

4. Verify your timer is active and running
```
systemctl status generate-index.timer
```
When it says that your timer is active and (waiting), that means your timer is running

You can also use `journalctl` command to check the logs for the service [[6]](#6systemdjournal)
```
journalctl -u generate-index.service
```
You have created your service and timer files!

## Task 3 Nginx Configuration
This entire section was made with the help of [Nginx ArchWiki](#7-nginx) and [Week 10 Notes](#8-week-twelve-notes)
1. Install Nginx
```
sudo pacman -S nginx
```
2. Start and enable the service
```
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```
Check if it is active and running with 
```
systemctl status nginx.service
```

3. Modify the nginx configuration file
```
sudo nvim /etc/nginx/nginx.conf
```
Here we have to configure the file so that it runs with our `webgen` user and that is serves our `index.html` file on port 80. To do this we first need to change `user` from `http` to `webgen`:
```bash
user webgen webgen
```
> This tells nginx to run the master process as `webgen`. This gives the configuration files the correct permissions to manage files in `webgen`'s directories

Add this line of code into you `http` section
```bash
http {
    include /etc/nginx/sites-enabled/*;
}
```
This helps load our other configuration files that we will make in the next steps

4. Create seperate server block directories
```
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled 
```

5. Create configuration file inside `sites-available` directory
```
sudo nvim /etc/nginx/sites-available/webgen.conf
```
Paste the following server block into the configuration file
```bash
server {
        # listening on port 80
        listen 80;
        # listening to HTTPS connections using IPv6
        listen [::]:80;

        # giving server a name
        server_name local.webgen;

        # specify where our content is
        root /var/lib/webgen/HTML;
        # specify the index html document
        index index.html;

        # specify what to do if a user requests our project root
                location / {
                        # checking for specific files, if none are found we return a 404 error
                        try_files $uri $uri/ =404;
                }
        }
```

6. Create a symlink to enable the site
```
sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sites-enabled/
```
7. Restart your nginx service
```
sudo systemctl retart nginx
```

You have configured your nginx server! 
> Some troubleshooting:

- Checking `webgen.conf` file for any errors: 
```
sudo nginx -t
```
- Checking the status of nginx:
```
systemctl status nginx
```
- Starting, enabling, restarting, stopping nginx:
```
systemctl start/enable/restart/stop nginx
```
- Checking journal log of service
```
sudo journalctl -u nginx
```

## Task 4 Setting up UFW Firewall
1. Install UFW
```
sudo pacman -S ufw
```

> ⚠️ **Warning:** Do not enable the UFW service yet because it will lock us out of our droplet. Please follow the next steps first.

2. Enable `ssh` and `http` connection [[8]](#8-week-twelve-notes)
```
sudo ufw allow SSH
sudo ufw allow http
```
> If successful, you should see `Rules updated` and `Rules updated (v6)` in your terminal

3. Enable ssh rate limiting [[9]](#9-uncomplicated-firewall)
```
sudo ufw limit SSH
```
> This denies connections from ip addresses that have attempted to intiate 6 or more connections in the last 30 seconds

4. Enable the service. This step should only be done after completing steps 2 & 3!
```
sudo ufw enable 
```
You can check your firewall status with:
```
sudo ufw status
```
Your firewall should looks something like this:
```
Status: active

To                         Action      From
--                         ------      ----
SSH                        LIMIT       Anywhere
80                         ALLOW       Anywhere
SSH (v6)                   LIMIT       Anywhere (v6)
80 (v6)                    ALLOW       Anywhere (v6)
```

You have activated your firewall!

# References
#### [1] Useradd Command
SS64, "Useradd Command," https://ss64.com/bash/useradd.html (accessed Nov. 22, 2024). 
#### [2] Mkdir Command
SS64, "Mkdir Command," https://ss64.com/bash/mkdir.html (accessed Nov. 22, 2024).
#### [3] Chown Command
SS64, "Chown Command," https://ss64.com/bash/chown.html (accessed Nov. 22, 2024).
#### [4] Systemd/Timers
ArchWiki, "Systemd/Timers," https://wiki.archlinux.org/title/Systemd/Timers (accessed Nov. 22, 2024).
#### [5] Systemctl
Arch Linux Manual Pages, "Systemctl(1)," https://man.archlinux.org/man/systemctl.1.en (accessed Nov. 22, 2024).
#### [6]Systemd/Journal
ArchWiki, "Systemd/Journal," https://wiki.archlinux.org/title/Systemd/Journal (accessed Nov. 22, 2024).
#### [7] Nginx
ArchWiki, "Nginx," https://wiki.archlinux.org/title/Nginx (accessed Nov. 23, 2024).
#### [8] Week Twelve Notes
GitLab, "Week Twelve Notes," https://gitlab.com/cit2420/2420-notes-f24/-/blob/main/2420-notes/week-twelve.md (accessed Nov. 23, 2024).
#### [9] Uncomplicated Firewall
ArchWiki, "Uncomplicated Firewall," https://wiki.archlinux.org/title/Uncomplicated_Firewall (accessed Nov. 23, 2024).
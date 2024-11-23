# Assignment-3
In this assignment we will be performing tasks such as creating new system users to run `.service` and `.timer` scripts on a Nginx server with a UFW firewall. 

## Task 1 Creating a System User
Here we will create a system user with a specified home directory and a login shell appropriate for a non-login user. 
1. Creating the system user with specified home and shell. [[1]](#1)
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

3. Creating the correct directory structure using `-p` option. [[2]](#2)
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

Finally we will give the user `webgen` ownership of the directory and its files [[3]](#3)
```
sudo chown -R webgen:webgen /var/lib/webgen
```
- `-chown`: used to change user ownership of files
- `-R`: option to recursively change ownership of directories and their contents
- `webgen:webgen /var/lib/webgen`: assign webgen and the group webgen to the specified directory 

You have finished creating a system user with the appropriate directory structure and ownership! 

# References
### [1]
SS64, "Useradd Command," https://ss64.com/bash/useradd.html (accessed Nov. 22, 2024). 
### [2]
SS64, "Mkdir Command," https://ss64.com/bash/mkdir.html (accessed Nov. 22, 2024).
### [3]
SS64, "Chown Command," https://ss64.com/bash/chown.html (accessed Nov. 22, 2024).

==Disclaimer:== The following guide uses Ubuntu 16.04 LTS as the server's OS. While these commands will likely work on any Debian-flavored distro, they may not work on others.


######Step 1: Create the server
Log in to your host's web interface and spin up a new server.

Depending on your host, when creating your server there might be an option to add your SSH public key. If you have that option, do it! You'll be able to log in to your server with a password or with an SSH key. While the password-based approach is fine to get things set up, longer term it's better to use SSH keys for authenticating.

If you don't know where your SSH public key is, you can usually find it in your user profile on your local computer: `/home/{username}/.ssh/id_rsa.pub` on Linux or `C:\Users\{username}\.ssh\id_rsa.pub` on Windows.

==Make sure you use the `.pub` file!== The `id_rsa` file is your *private* key and should never be shared. `.pub` for public.

If you don't have a SSH key yet, you need to generate one. [Here's a great guide from Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) on how to do so.


######Step 2: Save your server's IP to your hosts file (optional)
When your server is ready, make a note of the IP assigned to it. I like to put that IP in my `hosts` file so I don't need to remember it. Your `hosts` file is located at `/etc/hosts` on Linux or `C:\Windows\System32\drivers\etc\hosts` on Windows.

If your server's IP is `10.10.10.10` and you gave it the hostname `pluto`, add the following to your `hosts` file:

    10.10.10.10 pluto


######Step 3: SSH in to server
Now we can SSH in to our shiny new server!

    darren@laptop:~$ ssh root@pluto


######Step 4: Create a new user
Right away, I feel it's best to create a new user for yourself. The widely accepted best practice is to only log in as a non-root user and perform elevated operations with `sudo` or similar, rather than logging in as `root` to do everything.

I'm going to create my `darren` user:

    root@pluto:~# adduser darren

Choose a password and optionally enter your personal information. This will create the `darren` user and group, and will also create a home directory at `/home/darren/`.

Next we'll want to give your user as a `sudo` privileges. With default settings, adding `darren` to the `sudo` group should suffice:

    root@pluto:~# adduser darren sudo

Next, we need to copy your SSH public key into your user's home directory so we can SSH in as that user:

    root@pluto:~# mkdir /home/darren/.ssh
    root@pluto:~# cp /root/.ssh/authorized_keys /home/darren/.ssh/
    root@pluto:~# chmod 700 /home/darren/.ssh/
    root@pluto:~# chmod 600 /home/darren/.ssh/authorized_keys
    root@pluto:~# chown -R darren:darren /home/darren/.ssh/

Now everything should be ready for us to log out as `root` and log in as your user!

    root@rogue:~# exit
    logout
    Connection to pluto closed.

    darren@laptop:~$ ssh darren@pluto
    Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-31-generic x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/advantage

    12 packages can be updated.
    0 updates are security updates.


    *** System restart required ***
    Last login: Thu Aug 18 22:40:34 2016 from XX.XX.XX.XX

    darren@pluto:~$


######Step 5: Update server's SSH configuration
Now that we're logged in as our user, we should disable password-based authentication for SSH, and also disallow `root` from logging in over SSH.

Crack open your favorite terminal-based editor, I'll use `nano`:

    darren@pluto:~$ sudo nano /etc/ssh/sshd_config

We want to replace 2 lines:

- Replace `PermitRootLogin yes` with **`PermitRootLogin no`**
- Replace `#PasswordAuthentication yes` with **`PasswordAuthentication no`**

Note that we removed the `#` from the beginning of `#PasswordAuthentication`, which uncomments the setting.

After making changes to the `sshd_config` file, we need to restart the `ssh` service to have it reload the configuration:

    darren@pluto:~$ sudo systemctl restart ssh

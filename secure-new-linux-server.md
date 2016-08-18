Securing a new Linux server
===========================

1.  Create virtual machine in host's web interface

2.  Usually, you'll log in to your server via SSH with a password or with an SSH key.
    The password approach is fine to get things set up, but long-term it's better
    to use SSH keys for authenticating.

    When you create your server, there should be an option to add your SSH public key,
    which will get copied into the VM after it's created. Do it!
    The SSH key will be added to `/root/.ssh/authorized_keys` which will allow you to
    log in as `root` without having to type in a password.

    If you don't know where your SSH public key is, you can usually find it in
    your user profile on your local computer:
    * On Windows: `C:\Users\{username}\.ssh\id_rsa.pub`
    * On Linux: `/home/{username}/.ssh/id_rsa.pub`

    If you don't have a SSH key yet, you need to generate one.
    [Here's a great guide from Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
    on how to do so.

3.  When your VM is ready, make a note of the IP assigned to it.
    I like to put that IP in my `hosts` file so I don't need to remember it.
    Your `hosts` file is located:
    * On Windows: `C:\Windows\System32\drivers\etc\hosts`
    * On Linux: `/etc/hosts`

    If your VM's IP is `10.10.10.10` and you gave it the hostname `pluto`,
    add the following to your `hosts` file:
    ```
    echo 10.10.10.10 pluto
    ```

    If you have a domain name you're going to use with your server,
    you can utilize DNS to handle IP address resolution for you,
    and thus won't need to edit your `hosts` file.

3.  Now we can SSH into our shiny new server!
    ```
    darren@laptop:~$ ssh root@pluto
    ```

4.  Right away, I feel it's best to create a user for yourself.
    The widely accepted best practice is to only log in as yourself,
    and perform elevated operations with `sudo`,
    rather than logging in as `root` to do everything.
    ```
    root@pluto:~# adduser darren
    ```
    Choose a password and optionally enter your personal information.
    This will create the `darren` user and group,
    and will also create a home directory at `/home/darren/`.

5.  Next we'll want to add `darren` as a `sudoer`. With default settings,
    adding `darren` to the `sudo` group should suffice:
    ```
    root@pluto:~# adduser darren sudo
    ```

6.  Next, we need to copy your SSH public key into your new user's home directory
    so we can SSH in as that user:
    ```
    root@pluto:~# mkdir /home/darren/.ssh
    root@pluto:~# cp /root/.ssh/authorized_keys /home/darren/.ssh/
    root@pluto:~# chmod 700 /home/darren/.ssh/ && chmod 600 /home/darren/.ssh/authorized_keys
    root@pluto:~# chown -R darren:darren /home/darren/.ssh/
    ```
7.  Now everything should be ready for us to log out as `root` and log in as your user!
    ```
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
    ```
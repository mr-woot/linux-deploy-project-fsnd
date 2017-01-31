# Linux project deploying Full Stack Nanodegree.

#### 35.166.221.186 is my [PUBLIC\_IP] or [HOST\_IP] here
#### http://ec2-35-161-139-158.us-west-2.compute.amazonaws.com is my [HOST\_ADDRESS]
To check yours: [Visit here](http://www.hcidata.info/host2ip.cgi)

## Instructions for SSH access to the instance

* Download Private Key from [Udacity Developer Environment](https://www.udacity.com/account#!/development_environment)
* Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal. ``mv ~/Downloads/udacity_key.rsa ~/.ssh/``. **Note:** In my case environment's home directory is ``tushar@dell $`` (not the virtual environment like root@[PUBLIC_IP]).
* Open your terminal and type in ``chmod 600 ~/.ssh/udacity_key.rsa``
* In your terminal, type in ``ssh -i ~/.ssh/udacity_key.rsa root@35.166.221.186``

## Add grader user (from root's VM environment)
* ``sudo adduser grader``
* ``sudo touch /etc/sudoers.d/grader``
* ``sudo nano /etc/sudoers.d/grader``
* Add: ``grader ALL=(ALL:ALL) ALL``, ``Ctrl+X, Y, Enter`` to save in nano editor.
* **Note:** In terminal you will see something like ``root@ip-10-20-17-19:~$`` where ``ip-10-20-17-19`` is your local machine IP (different for different machines), add this IP to hosts file to avoid conflict of hosts. Edit ``sudo nano /etc/hosts``, append below ``127.0.0.1 localhost`` line ``127.0.0.1 localhost ip-10-20-17-19``. Save it.
* **Note:** When you first login using ssh, we used something like ``ssh -i ~/.ssh/udacity_key.rsa root@35.166.221.186``, where ``35.166.221.186`` is your [PUBLIC\_IP] but when you are logged in to the VM, you will see ``root@ip-10-20-17-19:~$`` where ``ip-10-20-17-19`` is your [LOCAL\_IP].

## Update and Upgrade the packages
* Run ``sudo apt-get update``, followed by ``sudo apt-get upgrade``.
* **Note:** You may want this to automate, ``sudo apt-get install unattended-upgrades`` followed by ``sudo dpkg-reconfigure -plow unattended-upgrades``.

## SSH Configurations (from grader's VM environment)
* Change SSH Port from SSH (port 22 to 2200), and only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
 * ``sudo nano /etc/ssh/sshd_config`` change port 22 to port 2200.
 * Change ``PermitRootLogin without-password`` to ``PermitRootLogin no`` to disallow root login.
 * Change PasswordAuthentication from no to yes. Change back after finishing SSH setup.
 * Append AllowUsers grader inside file to allow grader to login through SSH.
 * Save file and restart ssh service ``sudo service ssh reload``.
* Create SSH keys and copy to server manually.
 * On your local machine (in my case ``tushar@dell:$``), run ``ssh-keygen``, change file name if you want, default is ``id_rsa``, ``id_rsa.pub``, keys will be generated in local pc's ``~/.ssh`` directory.
 * Now, in grader's VM environment, create ``.ssh`` directory as ``sudo mkdir .ssh`` followed by ``sudo touch authorized_keys``. Copy the content of id_rsa.pub (in my case, linuxSetup.pub) from local machine into ``authorized_keys`` uisng ``sudo nano authorized_keys``.
 * Give permissions 644 to authorized_keys and 700 to .ssh using ``sudo chmod 700 ~/.ssh && sudo chmod 644 ~/.ssh/authorized_keys`` in grader's VM environment.
 * To login remotely into grader's account, ``ssh -i ~/.ssh/id_rsa grader@35.166.221.186`` (in my case, ``ssh -i ~/.ssh/linuxSetup grader@35.166.221.186``).

















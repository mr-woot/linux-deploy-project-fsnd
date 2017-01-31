# Linux project deploying Full Stack Nanodegree.

## Instructions for SSH access to the instance

1. Download Private Key from [Udacity Developer Environment](https://www.udacity.com/account#!/development_environment)
2. Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
``mv ~/Downloads/udacity_key.rsa ~/.ssh/``
3. Open your terminal and type in
``chmod 600 ~/.ssh/udacity_key.rsa``
4. In your terminal, type in
``ssh -i ~/.ssh/udacity_key.rsa root@35.166.221.186``

## Add grader user
1. ``sudo adduser grader``
2. ``sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/grader``
3. Edit as this: ``grader ALL=(ALL) NOPASSWD:ALL``

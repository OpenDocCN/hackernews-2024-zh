<!--yml
category: 未分类
date: 2024-05-27 14:51:00
-->

# Simulate an Ubuntu-like VM inside MacOS | by Yashodhan Mohan Bhatnagar | Medium

> 来源：[https://yashodhanmohan.medium.com/simulate-an-ubuntu-like-vm-inside-macos-2a20332e02e8](https://yashodhanmohan.medium.com/simulate-an-ubuntu-like-vm-inside-macos-2a20332e02e8)

# Simulate an Ubuntu-like VM inside MacOS

There are times, when you’re working on a remote system that has a Unix system like Ubuntu, RHEL, etc. You want to develop scripts, checkout if some packages exist, prototype command snippets or maybe test out some compatibility.

Anytime I’ve had to do that I had to spin up a t2 VM from AWS’s free tier and test it out and then close it. Always seemed a hassle especially since I needed Internet to simulate simple commands. It’s not like a system like Ubuntu is very far from what MacOS already has. But sometimes there are very subtle differences and wiggly bugs that absolutely need you to use the original OS. Sometimes simple commands like **date** have different flag behaviours.

Recently I had started using a Docker container of Ubuntu as a replacement of the t2 VM. For any testing, I would spin up the container, setup the dependeny packages, test out the commands, scripts, etc. and then let it be. Some times it would get killed, some days I absent mindedly cleaned up all the containers, once or twice I had to remove Docker runtime itself (for unrelated reasons). It would have been swell if I could treat that Docker container like my personal VM, which could maintain some amount of state. I don’t want to install basic packages like wget, curl everytime and officiating every minor change inside a Dockerfile was honestly a maintenance nightmare for me. If I created a symlink inside the OS filesystem, I didn’t want to codify it inside the Dockerfile.

To that end, I setup a batch of scripts that can simulate a VM, maintain state and let me treat the container as a VM. This is the story of “Friday”.

# State

There are two kinds of states I wanted to maintain —

1.  System state: This is the state that gets modified when I install packages, make system changes, add users, change permissions and do all kind of random admin stuff.
2.  Volume state: This was generally heavy duty stuff like files, CSVs, keys, etc that I don’t necessarily want to store in the container image. The files inside it could be small to large to very large. Taking it inside a container image would be, inconvenient, to say the least.

To handle the system state, I used a combination of **“docker export”** and **“docker import”** to routinely, and on-demand, backup the state of the container.

To handle the volume state, I simply mounted a directory from my MacOS filesystem into the container as “**/root**”. It has the additional side-effect that transferring files from my Mac to my “Ubuntu VM” is as simple as copying it from one directory to another. SFTP of the simplest kind.

# Friday

Friday is made of 4 simple scripts:

1.  ***start-friday.sh***
    This script checks if Friday is already running and if not, starts Friday, and takes an immediate backup of it’s state.
2.  ***freeze-friday.sh***
    This script is used by all other scripts and simply takes a backup of Friday’s state by running “**docker export**” to make a TGZ file and then an immediate “**docker import**” to bring it back into the docker image system.
3.  ***login-into-friday.sh***
    This script runs the docker exec command to enter Friday. The special magic spice is that when you exit the shell cleanly, it’ll take an immediate freeze of Friday. So this makes sure that if you’ve done any changes during your shell session, they’re not lost. Kind of like how applications ask you *“Do you want to save before quitting?”*
4.  ***stop-friday.sh*** This script takes a backup of Friday and stops it cleanly.

The core spice of Friday is the freeze-friday logic that ensures that each and every minor modification to Friday is backed up similar to how AWS maintains your EBS backups.

An additional protection that you can put around Friday is to setup **crontab** to take hourly/half-hourly backups of Friday by running freeze-friday.

I’ve put up the scripts below which only need you to modify the “ROOT_PATH” variable to a directory where Friday can store its TGZ backups and its Volume state. To setup, only once you need to pull your flavor or favorite OS’s base image from DockerHub or your registry to your local system and tag it as “***friday:latest***”.

***The scripts are available as a*** [***Gist here***](https://gist.github.com/yashodhanmohan/2c1b712883dec6dfba8ba2586df1b0cc)***.***

For instance I wanted my VM to be Ubuntu flavored, so:

setup

start-friday.sh

login-into-friday.sh

freeze-friday.sh

stop-friday.sh

You can also add freeze-friday to crontab like so:
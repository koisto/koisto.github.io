# PwnAdventure3 in 2022

## Introduction

I first heard about the [PwnAdventure](https://www.pwnadventure.com/) from [LiveOverflow's series](https://www.youtube.com/playlist?list=PLhixgUqwRTjzzBeFSHXrw9DnQtssdAwgG) on the hackable game. In short this is a game which designed to be exploited and was originally developed as part of a CTF.  

## Client and Server setup

It's 2022 and I decided that I wanted to explore the game and it's venerabilities for myself, the game was originally developed for Ghost in the Shellcode 2015 so I faced a few challnges getting everything to work. I am using Xubuntu 22.04.1.

To set up the server I followed the [instructions provided by LiveOverflow](https://github.com/LiveOverflow/PwnAdventure3#option-3---docker) to set the server up in a docker container. The only issue I had was that I needed to create a folder name `postrges-data` in the root of the repo to make the server run according to the instructions.

I had more difficulty getting the client to run. I downloaded the [Linux Client](https://www.pwnadventure.com/PwnAdventure3_Linux.zip), unzipped it and ran the binary as follows `./PwnAdventure3/Binaries/Linux/`. The first time I tried this I had issues with `libssl.so.1.0.0` being missing, this is an older version of libssl and is no longer available for download from Ubuntu's repositories. 

My first solution was to try and run the client in a VirtualBox VM using Ubuntu 14.04, this satisfied the libssl dependency but the client wouldn't run because VirtualBox doesn't support OpenGL3. I then attempted to run the game client in a docker container based on Ubuntu 14.04 and once again I ran into issues satisfying the graphics dependencies. I think with a little bit more persistance I could have succeeded with this approach. 

The final solution was to take libssl and libcrypto from the Ubuntu 14.04 VM I had created previously and copy them to the same directory as the game binary on my host machine. This meant that I avoided installing these older libraries system wide. Had I not previously created the VM I would have resorted to either downloading the missing libraries from elsewhere or buidling from source.

```
$ cd ~/PwnAdventure3_Client/PwnAdventure3/Binaries/Linux                                                                                 
$ ls -al
total 61168
drwxr-xr-x 2 james james     4096 Aug 13 14:40 .
drwxr-xr-x 3 james james     4096 May 19  2019 ..
-rw-r--r-- 1 james james  1926432 Aug 12 20:02 libcrypto.so.1.0.0
-rw-r--r-- 1 james james 10764815 May 19  2019 libGameLogic.so
-rw-r--r-- 1 james james   382984 Jun 20  2014 libssl.so.1.0.0
-rwxr-xr-x 1 james james 49541560 May 19  2019 PwnAdventure3-Linux-Shipping
```

It's also important to note that for the client to connect the server successfully:
- The server must be running
- The server.ini has to be configured correctly
- The client biary must be started from within the folder it is located in i.e. `cd ~/PwnAdventure3_Client/PwnAdventure3/Binaries/Linux` followed by `./PwnAdventure3-Linux-Shipping` or alternativly navigating to the folder in a file manager and then double clicking on the icon.



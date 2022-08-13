# PwnAdventure3 in 2022

## Introduction

I first heard about the [PwnAdventure](https://www.pwnadventure.com/) from [LiveOverflow's series](https://www.youtube.com/playlist?list=PLhixgUqwRTjzzBeFSHXrw9DnQtssdAwgG) on the hackable game. In short this is a game which designed to be exploited and was originally developed as part of a CTF.  

## Client and Server setup

It's 2022 and I decided that I wanted to explore the game and it's venerabilities for myself, the game was originally developed for Ghost in the Shellcode 2015 so I faced a few challnges getting everything to work. I am using Xubuntu 22.04.1.

To set up the server I followed the [instructions provided by LiveOverflow](https://github.com/LiveOverflow/PwnAdventure3#option-3---docker) to set the server up in a docker container. The only issue I had was that I needed to create a folder name `postrges-data` in the root of the repo to make the server run according to the instructions.

I had more difficulty getting the client to run. I downloaded the [Linux Client](https://www.pwnadventure.com/PwnAdventure3_Linux.zip), unzipped it and ran the binary as follows `./PwnAdventure3/Binaries/Linux/`. The first time I tried this I had issues with `libssl.so.1.0.0` being missing, this is an older version of libssl and is no longer available for download from Ubuntu's repositories. 

My first solution was to try and run the client in a virtualbox VM using Ubuntu 14.04, this satisfied the libssl dependency but the client wouldn't run because virtualbox doesn't support OpenGL3.  



---
title: Hosting your own Anki Sync Server
published: 2026-02-16
description: 'Removing the AnkiWeb dependency for syncing with Anki'
image: ''
tags:
- self-hosted
- anki
category: 'Anki'
draft: false
lang: ''
---
# Normie Version
In this example I will be hosting the server on my laptop with a docker container and syncing with my phone
1. Install docker desktop on your system
2. Install Rust (after installing, typing `rustc` -V in the terminal should show the version. This means that it is installed correctly)
3. Install git and clone the anki repo or Download the zip from github: https://github.com/ankitects/anki/archive/refs/heads/main.zip and extract.
4. Go to `anki/docs/syncserver` in the terminal. For example, if you have extracted in the Downloads folder (`cd /Users/username/Downloads/anki/docs/syncserver`)
5. Run  this command (builds image for ARM Macs and x86 Linux devices) 
```bash 
docker buildx build -f Dockerfile.distroless --platform linux/amd64,linux/arm64 --no-cache --build-arg ANKI_VERSION=25.09.2 -t anki-sync-server .
```
6. Run the container (here admin is the username and pass is the password, change to whatever you'd like): 
```bash
docker run --restart always -d -e "SYNC_USER1=admin:pass" -p 8080:8080 --mount type=volume,src=anki-sync-server-data,dst=/anki_data --name anki-sync-server
# (--restart always will start the container as soon as you start docker)(If you want to change the port, only change the first 8080 not the second. You'll have to change from 8080 to the number you gave below)
``` 

7. Check if the app is running with `docker logs -f -t anki-sync-server`
8. In the Anki desktop app, go to settings-> Syncing and provide this in the self hosted sync server http://localhost:8080
9. Go back to the main page and click the sync button and it should ask you for some credentials. Give the username and password that you set earlier (You do not need to make an AnkiWeb account)
10. Get the ip address of your computer (`ifconfig| grep inet`)
11. Repeat 8-9 in the mobile app but replace the url with http://your-computers-ip:8080 (eg: http://192.168.21.4:8080)
12. Try syncing, it should work. You can run the command in step 7 to see the progress.

# What I ended up hosting

With the announcement of the anking group taking over maintenance of Anki, it is important to migrate from using ankiweb to a self hosted alternative as I don't know what the new owner group will do with the project. For now, I have forked the anki repo and followed the instructions in the `anki/docs/syncserver` to build my own docker image of the application ( I picked the distroless option for security and ease of maintenance). I created a custom compose file which mirrors the docker run command found in the syncserver docs (Did a quick and dirty test with the server running on my mac and pinging it with netcat (`netcat -vz url:port`) to make sure it was exposed and then plugging in the url into the ios app) and it works! Now I need to provide better credentials and throw this behind my pangolin reverse proxy and I will have a globally accessible sync server for anki. However, I am still concerned about security because I am not able access the port when the container is configured to accept local connections only (something to troubleshoot - try setting it to local only and then giving the url in the ios app and syncing)

After building the image , the goal was to get it running on my homeserver which would always be available. I thought that Docker stores the images in a particular folder (which it does) but on macos it's in a volume that is abstracted away (You can apparently access the linux VM using `screen`) and an easier way to get the image is to use the `docker save` command. I ran the command, saved the image as a `.tar` file and then moved it to my homeserver with scp (The image was less than 100MB so this was not an issue).

When I tried to run the image on my homeserver, the next issue raised its' head was that the image was the wrong architecture. Docker, by default, builds images for the architecture that it was built on and without specifying the `--platform` flag with the platforms you want, it will not build for other platforms. After checking the documentation, one thing that I did notice was that I would have to create a new builder and use that but when checking the existing default builder with `docker buildx ls`, the platforms that I wanted (`linux/amd64`,`linux/arm64`) were already there so I just had to modify the existing build command with the `--platform` flag.

This was the final command:
`docker buildx build -f Dockerfile.distroless --platform linux/amd64,linux/arm64 --no-cache --build-arg ANKI_VERSION=25.09.2 -t anki-sync-server:multiarch .`

The last step was to add this to my reverse proxy so that I could access it from anywhere. I just added a new resource to pangolin and provided the exposed local port along with the path that I wanted.

# Building the image on linux
As per a comment left by the maintainer for my PR (#4562), I tried building the image on linux with the same platform flags. 

## Some observations and scratchpad stuff
 `--platform windows` did not work at all on linux with the build failing.

 `--platform linux/amd64,linux/arm64` worked after setting a custom builder using the docker-container runtime which supports multiplatform builds. The custom builder is the same one that is there from the docs.
 ```docker
 docker buildx create \
  --name container-builder \
  --driver docker-container \
  --bootstrap --use
  ```
  The build took 2290 seconds and did not use GPU. However, there was no output image and the build artefacts were entirely in the cache. Need to enable containerd as well.
Turn on containerd image store by adding below to `/etc/docker/daemon.json`
```
{
    "features": {
        "containerd-snapshotter": true
    }
}
```
Verify that it is working with `docker info -f '{{.DriverStatus }}'` should return containerd as the driver-type

This built the image but it did not work on my Mac with a command not found error. (I'm stupid, found out later that I need to use docker load and not docker import, try this again)
Installed docker desktop on linux and trying a build there. The resulting image works on mac with just docker save and docker load (no additional flags for either)

References:
1. https://docs.docker.com/build/building/multi-platform/
2. https://nitinayyagari.com/posts/building-multi-arch-docker-images/
3. https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/#tips-for-everyday-use
4. https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-using-a-repository
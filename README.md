# Docker Desktop versus Colima on Apple M2 silicon

So, this is written from the perspective of someone that doesn't really care too much which container engine that they're using. I simply wanted to evaluate a vendor's product and knew that containers would be the most efficient way to get up and running fast. I'm also undertaking this activity as a personal training exercise and that means that Docker Desktop licensing should not be an issue. However, if you are using containers in a work capacity but are not licensed for Docker Desktop then read on as this may provide you with viable open source solution for container management.

Also note that the challenges I faced here were specific to Apple MacOS users that are running on Apple's proprietary M1/M2 silicon - the irony is not lost on me here running on such expensive tin and concerned about relatively trivial licensing costs 😁.

## TLDR

If licensing is not an issue and you lack that `engineering curiousity` about the various container engines but simply want to get the job done [Docker Desktop](https://docs.docker.com/desktop/install/mac-install/) is the one for you!

Just don't forget to install Rosetta 2 as this will help with emulation of the containers that were built on Intel silicon.

``` shell
# softwareupdate --install-rosetta
```

And configure Docker Desktop to use Rosetta 2.
![Docker Engine Config](https://github.com/allthingsclowd/colimavdocker/assets/9472095/a4977577-15e4-4997-8680-d2d51e112355)

Then it's just business as usual with your Docker commands.

## Colima is a dropin replacement, kinda!
[Colima](https://github.com/abiosoft/colima) is another superb community project that is helping to improve the MacOS users' experience of managing containers on linux, MacOS and [Nix!?!](https://en.wikipedia.org/wiki/NixOS) (don't ask, I haven't played with Nix yet either).

It's as simple as
``` shell
# Homebrew
brew install colima
...
```
followed by
``` shell
colima start
...
```
and then you're good to go, right? Hmmm, yes, I suppose so, provided you're running some self contained (pun intented) images that don't require external mounts.

The challenge that I had was that I was running a relatively complex vendor application that required multiple containers and physical volume mounts on the underlying OS.
Also I had not yet optimised colima for the M2 silicon.

First things first, to optimise for M2 silicon and increase some of the basic resource configurations beyond their defaults let's kill this engine and make a few tweaks as follows.

``` shell
colima stop
...
colima delete
...
colima start --arch aarch64 --vm-type=vz --vz-rosetta --cpu 6 --memory 12 --disk 64
```
This should give you an optimal setup provided that you have similar resources available, if not then adjust to taste.

However, I still couldn't get the vendor application to successfully run. Remember, it worked first time on Docker Desktop.
Hunting through the docker container logs I could see the main container was stuck in an initialisation boot loop - it was complaining about file permissions on the mounted drives.

A quick search through the Colima Issues revealed the [challenge along with the workaround](https://github.com/abiosoft/colima/issues/83#issuecomment-1339269542) - at least this worked for me.

So the final working Colima configuration for my M2 MBP

- create or put the following in `/Users/<username>/.lima/_config/override.yaml`
``` shell
mountType: 9p
mounts:
  - location: "/Users/<username>"
    writable: true
    9p:
      securityModel: mapped-xattr
      cache: mmap
  - location: "~"
    writable: true
    9p:
      securityModel: mapped-xattr
      cache: mmap
  - location: /tmp/colima
    writable: true
    9p:
      securityModel: mapped-xattr
      cache: mmap
```
Please don't forget to swap out `<username>` for your own username above (if you do you're in good company but it was late in the evening for me, what's your excuse?)
- delete the existing engine and start again - this time swapping back to qemu emulation and the older mountType
``` shell
colima stop
...
colima delete
...
colima start --arch aarch64 --vm-type=qemu --vz-rosetta --cpu 6 --memory 12 --disk 64 --mount-type 9p
```
Finally we can successfully launch the same vendor application on the M2 silicon using Colima! Your standard Docker commands and `docker-compose.yml` files should all work as normal.

Docker's context switch is useful if you do decide to run multiple container engines. It tells the docker binary which container engine to talk to. Also note, when you start Docker Desktop it swaps the context over automatically (yes that does make sense, I guess).

``` shell
# docker context ls
NAME              DESCRIPTION                               DOCKER ENDPOINT                                  ERROR
colima            colima                                    unix:///Users/graz/.colima/default/docker.sock   
default           Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                      
desktop-linux *   Docker Desktop                            unix:///Users/graz/.docker/run/docker.sock
#
```

# Conclusion
In theory, the performace of Colima should be reduced as we are no longer using Apples native virtualisation by swapping back to QEMU in this instance.
I did feel that there was a slight reduction in the performance compared with Docker Desktop. 

Scientific Analysis - It's a beefy (technical term) application and consistently took 2 minutes from `docker compose up -d` to login on when running on Docker Desktop whereas it was taking 3 minutes plus to get the login prompt when running on Colima.

If you're in a rush, are a business, and can afford the license then Docker Desktop is possibly the best business focused option.
However, Engineers and Developers will definitely play with Colima and please do contribute towards the project - it works and is continually improving.

MBP Details -> Apple M2 Max, 32GB, OS 13.5.2
Docker Desktop -> version 4.22.0
Colima -> version 0.5.5

Happy Shipping!
Graz

https://github.com/allthingsclowd/colimavdocker/blob/e38ac283431ae9c7f8e2febed36799ea891a8e9b/docker-compose.yml#L1-L76




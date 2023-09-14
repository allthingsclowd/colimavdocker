# Docker Desktop versus Colima on Apple M2 silicon

So, this is written from the perspective of someone that doesn't really care too much which container engine that they're using. I simply wanted to evaluate a vendor's product and knew that containers would be the quickest way to get up and running fast. I'm also undertaking this activity as a personal training exercise and that means that Docker Desktop licensing should not be an issue. However, if you are using containers in a work capacity but are not licensed for Docker Desktop then read on as this may provide you with viable open source solution for container management.

Also note that the challenges I faced here were specific to Apple MacOS users that are running on Apple's proprietary M1/M2 silicon - the irony is not lost on me here running on such expensive tin and concerned about relatively trivial licensing costs :) .

## TLDR

If licensing is not an issue and you lack that `engineering curiousity` about the various container engines but simply want to get the job done [Docker Desktop](https://docs.docker.com/desktop/install/mac-install/) is the one for you!

Just don't forget to install Rosetta 2 as this will help with emulation of the containers that were built on Intel silicon.

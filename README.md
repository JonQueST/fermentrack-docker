# Fermentrack Docker Image

If you want to know more about Fermentrack please go to [https://docs.fermentrack.com/](https://docs.fermentrack.com/).

NOTE! This is still under testing/development and might not work as expected. 

In the future the version tag will be used to track the base installation, linux version, nginx, redis python etc. There will most likley be a new build every 3-6 months to include new security fixes from the core components. But you can always fork the repository and do your own build if you want.

- v0.1.0 = fermentrack release 4d8d89b from 22 Aug 2020 (testing)
- v0.2.0 = fermentrack release 4d8d89b from 22 Aug 2020 but with additional code to disable git integration (not to break the installation)
- v0.3.0 = fermentrack release 99495bf from 7 Nov 2020, most functions should work now (bluetooth is still not verified)
- v0.4.0 = fermentrack release 3f6a8a1 from 11 Nov 2020, locked version of numpy since the latest version gave wrong result in gravity calculation
- v0.5.0 = fermentrack release b4e7378 from 19 Dec 2020, tested bluetooth and firmware update

## Background

I wanted to move my installation to a normal x86 server in order to improve my possibilities to make backup of the database and brew logs. I have had one to many crashes where I lost most of the data.  

There is an official version of docker for fermentrack in developmetn but that takes a differnt approach than this one. The offical will be based on docker-compose and use multiple containers as one should design a docker application. My approach uses a single container since I dont have docker-compose support on my NAS which i use to runt the container, so for me this is a better approach. 

So I belive that there will be a need for both options based on what docker support your server/nas/workstation has. My intention is to keep this updated and as close to the offical docker setup of Fermentrack as possible. I'm have also been contributing to that project.

**The target for this image is a standard x86 linux host (not raspberry pi)** but if there is a need please put in a request. 

I looked at a few docker images for fermentrack (available on docker hub) but they where quite crude and just ran the installation scripts and most of them would not build. My approach was to base it on the manual installation steps I use for setting up the development environment. I have tried to mimic the normal installation procedure with a few exceptions in order to have a better fit towards docker. This is however my first attempt to create a docker build process so there are probably several improvements to be made.

I have modified the standard installation in the following way: 

- Database file (db.sqlite3) is moved to a subdirectory called /home/fermentrack/fermetrack/db in order to have a volume mount point (this is done since docker is not really good at handling a single file outside the container) which would be the case if the database file wasnt moved. 
- The django file secretsetings.py will now be copied from the mounted db directory into the container at start so that you can keep your own key. If it does not exist, one will be created for you. So dont delete it.
- Redis and Nginx will run as non root user for increased security. Port 8080 will be exposed inside the container since ports below 1024 requires root access. This is not a problem since we can transform that to any port outside the container. 
- Most functions in fermentrack should work, including GIT upgrades. There are a few things that will need to be tested in relation to bluetooth configuration that will require more changes to in the docker container.
- Serial connections will only work if docker is running on a linux host and the container is running in privliged mode.
- Validations have been added to check that data and db directories are mounted, otherwise it will not start. 
- Access rights on mounted volumes as well as database migrations are done before fermentrack starts at startup.
- During startup you can also see what git repo is used as source, linux kernel, nginx and redis versions and when docker image was built. 

## Installation

You can download the docker image from here .... Docker Hub under mpse2/fermentrack-docker

The following VOLUMES should be mounted for the container;

- YOUR PATH:/home/fermentrack/fermentrack/data
- YOUR PATH:/home/fermentrack/fermentrack/log
- YOUR PATH:/home/fermentrack/fermentrack/db
- /dev:/dev 

/dev is exported for the serial integration and firmware flashing. If you dont use these functions you can skip exporting volume mapping. it's required to export the whole catalog since fermentrack scans for the device that is attached. 

The following functions will require higher priviligies in order to work.

- Firmware flashing will require that the container is run in priviligied mode. 
- Bluetooth support requires both priviligied mode and network = "host". This means that the portmapping is ignored and fermentrack needs to be accessed via port 8080.

**Note that the db directory should contain db.sqlite3 and secretsetting.py**

The following PORTS should be mapped for the container (note that this is not used if network = host);

- YOUR PORT:8080

Any suggestions on improvements are welcome, and please note that this is not tested enough to ensure stability, please backup your data files before testing. I take no responsibility for lost data. The project is made available as is. 

Good luck!

## Troubleshooting

See Issues on github for known problems and options.

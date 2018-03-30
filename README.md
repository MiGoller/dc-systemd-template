# Running Docker services using docker-compose and systemd.
Have you ever wondered on how to start docker containers on startup of your system? 
## Running single Docker containter as a systemd service.
Well, it's easy to start a single container using systemd units. Just feel free to use the following template to do so and copy it to your personal service description folder or to `/etc/systemd/system/<Your service>.service`.
```
[Unit]
Description=<Your service description goes here>
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker run --name <Your container name> \
       <name of the image>

ExecStop=/usr/bin/docker stop -t 2 <Your container name>

ExecStopPost=/usr/bin/docker rm -f <Your container name>

[Install]
WantedBy=multi-user.target
```
Finally enable your new service.
```
$ sudo systemctl enable <path to your service file>
```
If you have placed your service file to something like `/etc/systemd/system/<Your service>.service` just enable the service by its filename of the service file.
```
$ sudo systemctl enable <servicename>
```
## Running multiple Docker container for a microservice environment using docker-compose and systemd.
But if you have to start more than one container for a service you will probably have to compose your service's container. Create service files for systemd units executing `docker-compose` to control the service's container and copy it to your personal service description folder or to `/etc/systemd/system/<Your service>.service`.
```
[Unit]
Description=<Your service description goes here>
Requires=docker.service
After=docker.service

[Service]
Restart=always
TimeoutStartSec=300

WorkingDirectory=<absolute path to the directory containing the docker-compose.yml file>

#   Remove old containers, images and volumes and update it
ExecStartPre=/usr/bin/docker-compose down
ExecStartPre=/usr/bin/docker-compose rm -f
#   Comment the following to not automatically update your images!
#ExecStartPre=/usr/bin/docker-compose pull

#   Run Compose up on service start.
ExecStart=/usr/bin/docker-compose up

#   Run Compose down, remove containers and volumes on service stop.
ExecStop=/usr/bin/docker-compose down

[Install]
WantedBy=multi-user.target
```
Finally enable your new service.
```
$ sudo systemctl enable <path to your service file>
```
If you have placed your service file to something like `/etc/systemd/system/<Your service>.service` just enable the service by its filename of the service file.
```
$ sudo systemctl enable <servicename>
```
## Creating docker-compose powered services using systemd templates
Do you know [Jay Alexander's "and it gets better"](http://www.sanfranciscomagictheater.com/)? Well, as far as I know, he has nothing to do with Docker at all, but I was so impressed when I saw him in San Fran, and I often remember one sentence: "And it gets better".
And that's the trick using a real systemd template file. For similar services it's easier to use a template file instead of creating a special service file for every service. But you'll have to create special files for services with special dependencies, etc. .
For similar service create a template file, lets call it `dc@.service` and save it to `/etc/systemd/system/dc@.service` with this content.
```
[Unit]
Description=%i service powered by docker compose
Requires=docker.service
After=docker.service

[Service]
Restart=always
TimeoutStartSec=300

#   Create a directory for each docker-composed service at /opt/dockerfiles.
#   Name the directory as the service.
WorkingDirectory=/opt/dockerfiles/%i

#   Remove old containers, images and volumes and update it
ExecStartPre=/usr/bin/docker-compose down
ExecStartPre=/usr/bin/docker-compose rm -f
#   Comment the following to not automatically update your images!
#ExecStartPre=/usr/bin/docker-compose pull

#   Run Compose up on service start.
ExecStart=/usr/bin/docker-compose up

#   Run Compose down, remove containers and volumes on service stop.
ExecStop=/usr/bin/docker-compose down

[Install]
WantedBy=multi-user.target
```
I suppose this setup. Check the pathes and modify to fit your needs, please.
* The Docker binaries are located at `/usr/bin/`.
* The Docker compose files are stored to individual folders for every service like `/opt/dockerfiles/<servicename>`.
### Create a new service based on the service template
First of all copy all the required content including the `docker-compose.yml` to the corresponding folder for service `/opt/dockerfiles/<servicename>`.
Now create the new service <servicename>.
```
$ sudo systemctl enable dc@<servicename>.service
```
That's it.

Enjoy to daemonize your Docker composed microservices.

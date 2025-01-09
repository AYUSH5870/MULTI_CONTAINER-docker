## ***About Docker***

"With Docker, developers can build any app in any language using any toolchain. “Dockerized” apps are completely portable and can run anywhere - colleagues’ OS X and Windows laptops, QA servers running Ubuntu in the cloud, and production data center VMs running Red Hat.

Developers can get going quickly by starting with one of the 13,000+ apps available on Docker Hub. Docker manages and tracks changes and dependencies, making it easier for sysadmins to understand how the apps that developers build work. And with Docker Hub, developers can automate their build pipeline and share artifacts with collaborators through public or private repositories.

Docker helps developers build and ship higher-quality applications, faster." -- What is Docker

Linux
The 3.10.x kernel is the minimum requirement for Docker.

MacOS
10.8 “Mountain Lion” or newer is required.

Windows 10
Hyper-V must be enabled in BIOS

VT-D must also be enabled if available (Intel Processors).

Windows Server
Windows Server 2016 is the minimum version required to install docker and docker-compose. Limitations exist on this version, such as multiple virtual networks and Linux containers. Windows Server 2019 and later are recommended.

Installation
Linux
Run this quick and easy install script provided by Docker:

curl -sSL https://get.docker.com/ | sh
If you're not willing to run a random shell script, please see the installation instructions for your distribution.

If you are a complete Docker newbie, you should follow the series of tutorials now.

macOS
Download and install Docker Community Edition. if you have Homebrew-Cask, just type brew install --cask docker. Or Download and install Docker Toolbox. Docker For Mac is nice, but it's not quite as finished as the VirtualBox install. See the comparison.

NOTE Docker Toolbox is legacy. You should to use Docker Community Edition, See Docker Toolbox.

Once you've installed Docker Community Edition, click the docker icon in Launchpad. Then start up a container:

docker run hello-world
That's it, you have a running Docker container.

If you are a complete Docker newbie, you should probably follow the series of tutorials now.

Windows 10
Instructions to install Docker Desktop for Windows can be found here

Once installed, open powershell as administrator and run:

# Display the version of docker installed:
docker version

# Pull, create, and run 'hello-world':
docker run hello-world
To continue with this cheat sheet, right click the Docker icon in the system tray, and go to settings. In order to mount volumes, the C:/ drive will need to be enabled in the settings to that information can be passed into the containers (later described in this article).

To switch between Windows containers and Linux containers, right click the icon in the system tray and click the button to switch container operating system Doing this will stop the current containers that are running, and make them unaccessible until the container OS is switched back.

Additionally, if you have WSL or WSL2 installed on your desktop, you might want to install the Linux Kernel for Windows. Instructions can be found here. This requires the Windows Subsystem for Linux feature. This will allow for containers to be accessed by WSL operating systems, as well as the efficiency gain from running WSL operating systems in docker. It is also preferred to use Windows terminal for this.

Windows Server 2016 / 2019
Follow Microsoft's instructions that can be found here

If using the latest edge version of 2019, be prepared to only work in powershell, as it is only a servercore image (no desktop interface). When starting this machine, it will login and go straight to a powershell window. It is reccomended to install text editors and other tools using Chocolatey.

After installing, these commands will work:

# Display the version of docker installed:
docker version

# Pull, create, and run 'hello-world':
docker run hello-world
Windows Server 2016 is not able to run Linux images.

Windows Server Build 2004 is capable of running both linux and windows containers simultaneously through Hyper-V isolation. When running containers, use the --isolation=hyperv command, which will isolate the container using a seperate kernel instance.

Check Version
It is very important that you always know the current version of Docker you are currently running on at any point in time. This is very helpful because you get to know what features are compatible with what you have running. This is also important because you know what containers to run from the docker store when you are trying to get template containers. That said let see how to know which version of docker we have running currently.

docker version shows which version of docker you have running.
Get the server version:

$ docker version --format '{{.Server.Version}}'
1.8.0
You can also dump raw JSON data:

$ docker version --format '{{json .}}'
{"Client":{"Version":"1.8.0","ApiVersion":"1.20","GitCommit":"f5bae0a","GoVersion":"go1.4.2","Os":"linux","Arch":"am"}
Containers
Your basic isolated Docker process. Containers are to Virtual Machines as threads are to processes. Or you can think of them as chroots on steroids.

Lifecycle
docker create creates a container but does not start it.
docker rename allows the container to be renamed.
docker run creates and starts a container in one operation.
docker rm deletes a container.
docker update updates a container's resource limits.
Normally if you run a container without options it will start and stop immediately, if you want keep it running you can use the command, docker run -td container_id this will use the option -t that will allocate a pseudo-TTY session and -d that will detach automatically the container (run container in background and print container ID).

If you want a transient container, docker run --rm will remove the container after it stops.

If you want to map a directory on the host to a docker container, docker run -v $HOSTDIR:$DOCKERDIR. Also see Volumes.

If you want to remove also the volumes associated with the container, the deletion of the container must include the -v switch like in docker rm -v.

There's also a logging driver available for individual containers in docker 1.10. To run docker with a custom log driver (i.e., to syslog), use docker run --log-driver=syslog.

Another useful option is docker run --name yourname docker_image because when you specify the --name inside the run command this will allow you to start and stop a container by calling it with the name the you specified when you created it.

Starting and Stopping
docker start starts a container so it is running.
docker stop stops a running container.
docker restart stops and starts a container.
docker pause pauses a running container, "freezing" it in place.
docker unpause will unpause a running container.
docker wait blocks until running container stops.
docker kill sends a SIGKILL to a running container.
docker attach will connect to a running container.
If you want to detach from a running container, use Ctrl + p, Ctrl + q. If you want to integrate a container with a host process manager, start the daemon with -r=false then use docker start -a.

If you want to expose container ports through the host, see the exposing ports section.

Restart policies on crashed docker instances are covered here.

CPU Constraints
You can limit CPU, either using a percentage of all CPUs, or by using specific cores.

For example, you can tell the cpu-shares setting. The setting is a bit strange -- 1024 means 100% of the CPU, so if you want the container to take 50% of all CPU cores, you should specify 512. See https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/#_cpu for more:

```
docker run -it -c 512 agileek/cpuset-test
```
You can also only use some CPU cores using cpuset-cpus. See https://agileek.github.io/docker/2014/08/06/docker-cpuset/ for details and some nice videos:

```
docker run -it --cpuset-cpus=0,4,6 agileek/cpuset-test
```
Note that Docker can still see all of the CPUs inside the container -- it just isn't using all of them. See moby/moby#20770 for more details.

Memory Constraints
You can also set memory constraints on Docker:

docker run -it -m 300M ubuntu:14.04 /bin/bash
Capabilities
Linux capabilities can be set by using cap-add and cap-drop. See https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities for details. This should be used for greater security.

To mount a FUSE based filesystem, you need to combine both --cap-add and --device:

docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse sshfs
Give access to a single device:

docker run -it --device=/dev/ttyUSB0 debian bash
Give access to all devices:

docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb debian bash
More info about privileged containers here.

### Info
```
docker ps shows running containers.
docker logs gets logs from container. (You can use a custom log driver, but logs is only available for json-file and journald in 1.10).
docker inspect looks at all the info on a container (including IP address).
docker events gets events from container.
docker port shows public facing port of container.
docker top shows running processes in container.
docker stats shows containers' resource usage statistics.
docker diff shows changed files in the container's FS.
docker ps -a shows running and stopped containers.
```
docker stats --all shows a list of all containers, default shows just running.

### Import / Export
```
docker cp copies files or folders between a container and the local filesystem.
```
```
docker export turns container filesystem into tarball archive stream to STDOUT.
```
Executing Commands
```
docker exec to execute a command in container.
```
To enter a running container, attach a new shell process to a running container called foo, use: docker exec -it foo /bin/bash.

**Images**
Images are just templates for docker containers.

Lifecycle
**docker images-** shows all images.
**docker import-** creates an image from a tarball.
**docker build </tagname> -</dockerr repo> .** creates image from Dockerfile.
**docker commit-** creates image from a container, pausing it temporarily if it is running.
**docker rmi-** removes an image.
**docker load <img id>-** loads an image from a tar archive as STDIN, including images and tags (as of 0.7).
**docker save-** saves an image to a tar archive stream to STDOUT with all parent layers, tags & versions (as of 0.7).
Info
**docker history-** shows history of image.
**docker tag <>-** tags an image to a name (local or registry).
Cleaning up
While you can use the docker rmi command to remove specific images, there's a tool called docker-gc that will safely clean up images that are no longer used by any containers. As of docker 1.13, docker image prune is also available for removing unused images. See Prune.

Load/Save image
Load an image from file:

docker load < my_image.tar.gz
Save an existing image:

docker save my_image:my_tag | gzip > my_image.tar.gz
Import/Export container
Import a container as an image from file:

cat my_container.tar.gz | docker import - my_image:my_tag
Export an existing container:

docker export my_container | gzip > my_container.tar.gz
Difference between loading a saved image and importing an exported container as an image
Loading an image using the load command creates a new image including its history.
Importing a container as an image using the import command creates a new image excluding the history which results in a smaller image size compared to loading an image.

Networks
Docker has a networks feature. Docker automatically creates 3 network interfaces when you install it (bridge, host none). A new container is launched into the bridge network by default. To enable communication between multiple containers, you can create a new network and launch containers in it. This enables containers to communicate to each other while being isolated from containers that are not connected to the network. Furthermore, it allows to map container names to their IP addresses. See working with networks for more details.

Lifecycle
docker network create NAME Create a new network (default type: bridge).
docker network rm NAME Remove one or more networks by name or identifier. No containers can be connected to the network when deleting it.
Info
docker network ls List networks
docker network inspect NAME Display detailed information on one or more networks.
Connection
docker network connect NETWORK CONTAINER Connect a container to a network
docker network disconnect NETWORK CONTAINER Disconnect a container from a network
You can specify a specific IP address for a container:

# create a new bridge network with your subnet and gateway for your ip block
docker network create --subnet 203.0.113.0/24 --gateway 203.0.113.254 iptastic

# run a nginx container with a specific ip in that block
$ docker run --rm -it --net iptastic --ip 203.0.113.2 nginx

# curl the ip from any other place (assuming this is a public ip block duh)
$ curl 203.0.113.2
Registry & Repository
A repository is a hosted collection of tagged images that together create the file system for a container.

A registry is a host -- a server that stores repositories and provides an HTTP API for managing the uploading and downloading of repositories.

Docker.com hosts its own index to a central registry which contains a large number of repositories. Having said that, the central docker registry does not do a good job of verifying images and should be avoided if you're worried about security.

docker login to login to a registry.
docker logout to logout from a registry.
docker search searches registry for image.
docker pull pulls an image from registry to local machine.
docker push pushes an image to the registry from local machine.
Run local registry
You can run a local registry by using the docker distribution project and looking at the local deploy instructions.

Also see the mailing list.

Dockerfile
The configuration file. Sets up a Docker container when you run docker build on it. Vastly preferable to docker commit.

Here are some common text editors and their syntax highlighting modules you could use to create Dockerfiles:

If you use jEdit, I've put up a syntax highlighting module for Dockerfile you can use.
Sublime Text 2
Atom
Vim
Emacs
TextMate
VS Code
Also see Docker meets the IDE
Instructions
.dockerignore
FROM Sets the Base Image for subsequent instructions.
MAINTAINER (deprecated - use LABEL instead) Set the Author field of the generated images.
RUN execute any commands in a new layer on top of the current image and commit the results.
CMD provide defaults for an executing container.
EXPOSE informs Docker that the container listens on the specified network ports at runtime. NOTE: does not actually make ports accessible.
ENV sets environment variable.
ADD copies new files, directories or remote file to container. Invalidates caches. Avoid ADD and use COPY instead.
COPY copies new files or directories to container. By default this copies as root regardless of the USER/WORKDIR settings. Use --chown=<user>:<group> to give ownership to another user/group. (Same for ADD.)
ENTRYPOINT configures a container that will run as an executable.
VOLUME creates a mount point for externally mounted volumes or other containers.
USER sets the user name for following RUN / CMD / ENTRYPOINT commands.
WORKDIR sets the working directory.
ARG defines a build-time variable.
ONBUILD adds a trigger instruction when the image is used as the base for another build.
STOPSIGNAL sets the system call signal that will be sent to the container to exit.
LABEL apply key/value metadata to your images, containers, or daemons.
SHELL override default shell is used by docker to run commands.
HEALTHCHECK tells docker how to test a container to check that it is still working.
Tutorial
Flux7's Dockerfile Tutorial
Examples
Examples
Best practices for writing Dockerfiles
Michael Crosby has some more Dockerfiles best practices / take 2.
Building Good Docker Images / Building Better Docker Images
Managing Container Configuration with Metadata
How to write excellent Dockerfiles
Layers
The versioned filesystem in Docker is based on layers. They're like git commits or changesets for filesystems.

Links
Links are how Docker containers talk to each other through TCP/IP ports. Atlassian show worked examples. You can also resolve links by hostname.

This has been deprecated to some extent by user-defined networks.

NOTE: If you want containers to ONLY communicate with each other through links, start the docker daemon with -icc=false to disable inter process communication.

If you have a container with the name CONTAINER (specified by docker run --name CONTAINER) and in the Dockerfile, it has an exposed port:

EXPOSE 1337
Then if we create another container called LINKED like so:

docker run -d --link CONTAINER:ALIAS --name LINKED user/wordpress
Then the exposed ports and aliases of CONTAINER will show up in LINKED with the following environment variables:

$ALIAS_PORT_1337_TCP_PORT
$ALIAS_PORT_1337_TCP_ADDR
And you can connect to it that way.

To delete links, use docker rm --link.

Generally, linking between docker services is a subset of "service discovery", a big problem if you're planning to use Docker at scale in production. Please read The Docker Ecosystem: Service Discovery and Distributed Configuration Stores for more info.

Volumes
Docker volumes are free-floating filesystems. They don't have to be connected to a particular container. You can use volumes mounted from data-only containers for portability. As of Docker 1.9.0, Docker has named volumes which replace data-only containers. Consider using named volumes to implement it rather than data containers.

Lifecycle
docker volume create
docker volume rm
Info
docker volume ls
docker volume inspect
Volumes are useful in situations where you can't use links (which are TCP/IP only). For instance, if you need to have two docker instances communicate by leaving stuff on the filesystem.

You can mount them in several docker containers at once, using docker run --volumes-from.

Because volumes are isolated filesystems, they are often used to store state from computations between transient containers. That is, you can have a stateless and transient container run from a recipe, blow it away, and then have a second instance of the transient container pick up from where the last one left off.

See advanced volumes for more details. Container42 is also helpful.

You can map MacOS host directories as docker volumes:

```
docker run -v /Users/wsargent/myapp/src:/src
```
You can use remote NFS volumes if you're feeling brave.

You may also consider running data-only containers as described here to provide some data portability.

Be aware that you can mount files as volumes.

Exposing ports
Exposing incoming ports through the host container is fiddly but doable.

This is done by mapping the container port to the host port (only using localhost interface) using -p:

```
docker run -p 127.0.0.1:$HOSTPORT:$CONTAINERPORT \--name CONTAINER \-t someimage
```
You can tell Docker that the container listens on the specified network ports at runtime by using EXPOSE:

EXPOSE <CONTAINERPORT>
Note that EXPOSE does not expose the port itself - only -p will do that.

To expose the container's port on your localhost's port, run:

```
docker -iptables -t nat -A DOCKER -p tcp --dport <LOCALHOSTPORT> -j DNAT --to-destination <CONTAINERIP>:<PORT>
```
If you're running Docker in Virtualbox, you then need to forward the port there as well, using forwarded_port. Define a range of ports in your Vagrantfile like this so you can dynamically map them:

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  ...

  (49000..49900).each do |port|
    config.vm.network :forwarded_port, :host => port, :guest => port
  end

  ...
end
If you forget what you mapped the port to on the host container, use docker port to show it:

docker port CONTAINER $CONTAINERPORT

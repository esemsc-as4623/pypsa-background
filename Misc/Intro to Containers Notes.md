2026-01-26

GitHub repo: https://github.com/ImperialCollegeLondon/RCDS-intro-to-containers

```bash
docker --version

docker ps # check the running containers
docker ps -a # check all containers

docker run <IMAGE-NAME> # run a container image

docker images # check the images

docker rm <CONTAINER-ID> # remove container by ID
docker rmi <IMAGE-ID> # remove image by ID
# remove container before removing image
# use first 3 characters of ID

docker rm $(docker ps -a -q) # remove all container instances
docker rmi $(docker image ls -q) # remove all container images

docker run -t -i debian:stable-slim "/bin/bash"
# -t = allocate pseudo-tty to access container
# -i = interactive mode
# debian:stable-slim (or any OS image)
# "/bin/bash" = entry point
# --entrypoint "/bin/bash" = to override default entrypoint
# #/exit = to exit container

docker run -t -d debian:stable-slim
# -d = detach container and run in background
dcoker exec -i -t <CONTAINER-ID> <ENTRYPOINT>
# enter container image that is already running in interactive mode
docker stop <CONTAINER-ID> # stop the detached container

docker run --rm -t -i -v ${PWD}:/data debain:stable-slim "/bin/bash"
# --rm = remove container instance after running
# -v = mount a local directory
# use -v multiple times for multiple directories
# ${PWD} = path of working directory (in MacOS)
# :/data = path in container following entrypoint
# any edits in the mounted volumes will be reflected locally

docker network create backend
# create network to allow database system and interface to communicate
docker network ls # check network
docker run -d --name <CONTAINER-NAME> --network <NETWORK-NAME> -e <ENV-VAR>=<ENV-VALUE> -v<LOCAL-DIR>:<HOST-DIR> <IMAGE-NAME:TAG>
# -e = add environment variables, e.g. POSTGRES_USER=xxxx
# e.g. docker run -d --name database --network backend -e POSTGRES_USER=hackmd -e POSTGRES_PASSWORD=hackmdpass -e POSTGRES_DB=hackmd -v ${PWD}/database:/var/lib/postgresql/data postgres:9.6-alpine

docker run -d --name <CONTAINER-NAME> --network <NETWORK-NAME> -e <ENV-VAR>=<ENV-VALUE> -p <HOST-PORT>:<CONTAINER-PORT> --restart always --link <CONTAINER-NAME> <CREATOR/IMAGE-NAME:TAG>
# -p = mapping port from host to container instance
# --restart alwars = in case it crashes, restarts
# --link = to link another container
# e.g. docker run -d --name app --network backend -e HMD_DB_URL=postgres://hackmd:hackmdpass@database:5432/hackmd -p 3000:3000 --restart always --link database hackmdio/hackmd:1.2.0

docker run -it --rm -p 8006:8006 --device=/dev/kvm --device=/dev/net/tun --cap-add NET_ADMIN --stop-timeout 120 dockurr/windows
# --device=/dev/kvm = improve performance with KVM acceleration
# --device=/dev/net/tun = container to use TUN (network tunnel) device, often required for VPNs
# --cap-add NET_ADMIN = create bridge connection from host to container
  
# for ubuntu images
export DEBIAN_FRONTEND=noninteractive # to disable country/region selection
apt update && apt install <PACKAGE>
```

To publish, use GitHub Codespaces
1. create `devcontainer.json`
```json
{
	"name": "Minimal Docker in Codespaces",
	"image": "mcr.microsoft.com/vscode/devcontainers/base:ubuntu-22.04",
	"features": {
		"ghcr.io/devcontainers/features/docker-in-docker:2": {
			"version": "latest",
			"enableNonRootDocker": true,
			"moby": true
			}
		},
	// run after creation (can be a script)
	"postCreateCommand": "docker --version",
	"remoteUser": "vscode"
}
```
2. update with application requirements

Dockerfile instructions:
1. `FROM` = initialize build specify base image
	- contains OS
2. `RUN` = run a command
3. `ADD` = copy files, directories, etc. add to filesystem of image at the path `.`
	- `ADD` VS `COPY`? `ADD` will auto-unzip files and can auto-download files from remote servers
4. `CMD` = default computational work performed when container is executed using `docker run`
	- max. one `CMD` per Dockerfile
5. `ENTRYPOINT` = configure container that will run as an executable
	- set image's main command, allowing image to be run as though it was that command; then use `CMD` as default flags
	- more useful when you want to run and plug in external params (e.g. ML pre-trained weights, data-agnostic workflow, sys args)
6. `WORKDIR` = set working directory for any of the above instructions
	- will be created even if not used in any subsequent instruction
7. `MAINTAINER <NAME-OF-PROJECT> (<CONTACT-DETAILS>)` = add contact details

to build container image:
1. create Dockerfile

```Dockerfile
FROM ubuntu
ADD print.sh ./
RUN chmod a+x ./print.sh 
CMD ["./print.sh"]
```
2. `cd` into same directory as Dockerfile
3. `docker build -t <IMAGE-NAME:TAG> .`
4. check that it appears in `docker images`
5. run: `docker run <IMAGE-NAME>`

multi-platform image build:
```bash
# optional to turn on container image storage
# (otherwise it won't show up in docker images)
docker buildx create --name=container --driver=docker-container --use --bootstrap

docker buildx build --platform=linux/amd64,linux/arm64 -t <IMAGE-NAME> .
```
- if the local Docker engine doesn't have the capacity for multi-platform image builds, push to Docker hub
```bash
docker buildx build --platform=linux/amd64,linux/arm64 -t <DOCKERHUB-USERNAME>/<IMAGE-NAME> --push .
```
- try `docker login` to check username
- default builder uses local Docker daemon
- buildx runs buildkit inside container

Python Dockerfile
```Dockerfile
FROM ubuntu
WORKDIR /usr/local/src
ADD requirements.txt ./
RUN apt update && apt install -y python3 python3-pip python3-venv
# create non-root user
RUN useradd -m -s /bin/bash appuser && chown -R appuser:appuser /usr/local/src

# switch to the user added
USER appuser
RUN python3 -m venv /home/appuser/venv && \
# --no-cache-dir remove cache files after installation
    /home/appuser/venv/bin/pip install --no-cache-dir -r requirements.txt
# --break-system-packages avoids errors from system backend
ADD test.py ./

ENTRYPOINT ["/home/appuser/venv/bin/python3", "py_test.py"]
```

C++ Dockerfile
```Dockerfile
FROM ubuntu
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && \
    apt-get -y install g++ mono-mcs && \
    rm -rf /var/lib/apt/lists/*
WORKDIR /usr/local/src
# external params used by the scripts
ENV NAME VAR1 VAR2 VAR3
ENV OUT_DIR=/output_dir/
ADD run_helloC.sh ./
ADD helloworld.cpp ./
RUN g++ -o helloworld helloworld.cpp

CMD ["/bin/sh", "./run_helloC.sh"]
```

```Dockerfile
# add build-stage name
FROM ubuntu:20.04 AS build
WORKDIR /app
RUN apt-get update && apt-get install -y \
    build-essential
    # remove apt cache from apt-get
    && rm -rf /var/lib/apt/lists/*
COPY main.cpp ./
RUN g++ -o myprogram main.cpp

# second stage runs app
FROM ubuntu:20.04

# install C shared libraries to run
# RUN apt-get update && apt-get install -y ...

# copy the compiled application binary into the final image
COPY --from=build /app/myprogram /usr/local/bin

# set the standard image metadata to run it
CMD ["myprogram"]
```

release disk space after building container images
`docker builder prune -a`
build a docker image without cache
`docker build --no-cache -t my-image:latest .`

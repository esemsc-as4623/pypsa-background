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

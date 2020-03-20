# Docker Deep Dive - Highlights

- "Press Ctrl-PQ to exit the container without terminating it." (pg. 55)
- "You can attach your shell to the terminal of a running container with the `docker container exec` command." (pg. 56)
- "The local image repository on a Linux-based Docker host is usually located at `var/lib/docker/<storage-driver>. On Windows-based Docker hosts this is `C:\ProgramData\docker\windowsfilter." (pg. 79)
- "Docker images are stored in *image registries*. The most common registry is Docker Hub (<https://hub.docker.com>)" (pg. 82)
- "Image registries contain multiple *image repositories*. In turn, image repositories can contain multiple images." (pg. 82)
- "...*official repositories* contain images that have been vetted by Docker, Inc." (pg. 83)
- "The format for `docker image pull`, when working with an image from an official repository is: `docker image pull <repository>:<tag>` (pg. 84)
- "...if you **do not** specify an image tag after the repository name, Docker will assume you are referring to the image tagged as `latest`." (pg. 84)
- "If you want to pull images from 3rd party registries (not Docker Hub), you need to prepend the repository name with the DNS name of the registry. For example, if the image in the example above was in the Google Container Registery (GCR) you'd need to add `gcr.io` beofre the repository name..." (pg. 86)
- "Pull all of the images in a repository by adding the `-a` flag to [the] `docker image pull` command." (pg. 86)
- "Docker provides the `--filter` flag to filter the list of images returned by `docker image ls`." (pg. 87)
- "A dangling image is an image that is no longer tagged, and appears in listings as `<none>:<none>`. A common way they occur is when building a new image and tagging it with an existing tag." (pg. 87)
- "You can delete all dangling images on a system with the `docker image prune` command. If you add the `-a` flag, Docker will also remove all unused images (those not in use by containers)." (pg. 88)
- "The `docker search` command lets you search Docker Hub from the CLI." (pg. 89)
- "You can use `--filter "is-official=true"` so that only official repos are displayed." (pg 90)
- "By default, Docker will only display 25 lines of results. However, you can use the `--limit` flag to increase that to a maximum of 100." (pg. 90)
- Another way to see the layers of an image is to inspect the image with the `docker image inspect` command." (pg. 92)
- "..the file in the higher layer obscures the file directly below it. This allows updated versions of files to be added as new layers to the image." (pg. 95)
- "...all images now get a cryptographic content hash." (pg. 98)
- "...each layer also gets something called a *distribution hash*. This is a hash of the compressed version of the layer." (pg. 100)
- "...all *official images* have manifest lists." (pg. 103)
- "When you no longer need an image, you can delete it from your Docker host with the `docker image rm` command. `rm` is short for remove." (pg. 103)
- "...if an image layer is shared by more than one image, that layer will not be deleted until all images that reference it have been deleted." (pg. 103)
- "A handy shortcut for **deleteing all images** on a Docker host is to run the `docker image rm` command and pass it a list of all image IDs on the system by calling `docker image ls` with the `-q` flag... `$ docker image rm $(docker image ls -q) -f`" (pg. 104)
- "...you tell it an image to use and an app to run: `docker container run <image> <app>.`" (pg. 108)
- "The `-it` flags will connect your current terminal window to the container's shell." (pg. 108)
- "" (pg. 111)

Page 111

>At a high level, we can say that hypervisors perform **hardware virtualization** - they carve up physical hardware resources into virtual versions. ON the other hand, containers perform **OS virtualization** - they carve up OS resources into virtual versions.

>...every OS consumes a slice of CPU, a slice of RAM, a slice of storage etc. Most need their own licenses as well as people and infrastructure to patch and upgrade them. Each OS also presents a sizable attack surface. We often refer to all of this as the _**OS tax**_ or the _**VM tax**_ - every OS you install consumes resources!

> Because a container isn't a full-blown OS, it starts **much faster** than a VM.

Page 115

> `docker container run <options> <image>:<tag> <app>`.

>The long number after the `@` is the first 12 characters of the container's unique ID.

Page 117

> ...a container cannot exist without a running process...

> This is also true of Windows containers - **killing the main process in the container will also kill the container**

> ... you can re-attach your terminal to [the container] with the `docker container exec` command.

> `docker container exec -it <container-name-or-ID> pwsh.exe`

> ... the `docker container exec` command created a new Bash or PowerShell process and attached to that.

Page 118

> It's a common myth that containers can't persist data. They can!

Page 121

> ...it's considered best practice to take the two-step approach of stopping the container first and then deleting it.

> ... if you're storing container data in a *volume*, that data's going to persist even after the container is gone.

Page 123

> The **always** policy is the simplest. It will always restart a stopped container unless it has been explicitly stopped, such as via a `docker container stop` command.

Page 124

> An interesting feature of the `--restart always` policy is that a stopped container will be restarted when the Docker daemon starts.

> The main difference between the **always** and the **unless-stopped** policies is that containers with the `--restart unless-stopped` policy will not be restarted when the daemon restarts if they were in the `Stopped (Exited)` state.

Page 126

> The **on-failure** policy will restart a container if it exits with a non-zero exit code. It will also restart containers when the Docker daemon restarts, even containers that were in the stopped state.

Page 127

> ... started this container in the background with the `-d` flag."

> `-d` stands for **d**aemon mode, and tells the container to run in the background.

> `-p 80:8080`. The `-p` flag maps the ports on the Docker host to ports inside the container. This time we're mapping port 80 on the Docker host to port 8080 inside the container. This means that traffic hitting the Docker host on port 80 will be directed to port 8080 inside of the container.

Page 133

> The process of taking an application and configuring it to run as a container is called "containerizing." Sometimes we call it "Dockerizing."

Page 136

> The directory containing the application is referred to as the *build context*. It's a common practice to keep your Dockerfile in the root directory of the *build context*. It's also important that **Dockerfile** starts with a capital "**D**" and is all one word. "dockerfile" and "Docker file" are not valid.

> Do not underestimate the impact of the Dockerfile as a form of documentation!

Page 138

> It's considered a best practice to list a maintainer of an image so that other potential users have a point of contact when working with it.

Page 142

> The format of the command is `docker image tag <current-tag> <new-tag> and it adds an additional tag, it does not overwrite the original.

Page 145

> All non-comment lines are **Instructions**. Instructions take the format `INSTRUCTION argument`. Instruction names are not case snesitive, but it is normal practice to write them in UPPERCASE.

> Examples of instructions that create new layers are `FROM`, `RUN`, and `COPY`. Examples of instructions that create metadata include `EXPOSE`, `WORKDIR`, `ENV`, and `ENTRYPOINT`.

> ...if an instruction is adding *content* such as files and programs to the image, it will create a new layer.

Page 150

> `COPY --from` instructions are used to **only copy production-related application code** from the images built by the previous stages. They do not copy across build artifacts that are not needed for production.

Page 154

> You can force the build process to ignore the entire cache by passing the `--no-cache=true` flag to the `docker image build` command.

> To protect against [using cached content when there are changes], Docker performs a checksum against each file being copied, and compares that to a checksum of the same file in the cached layer. If the checksums do not match, the cache is invalidated and a new layer is built.

> Add the `--squash` flag to the `docker image build` command if you want to create a squashed image.

Page 155

> If you are building LInux images, and using the apt package manager, you should use the `no-install-recommends` flag with the `apt-get install` command. This makes sure that `apt` only installs main dependencies (packages in the `Depends` field) and not recommended or suggested packages. This can greatly reduce the number of unwanted packages that are downloaded into your images.

> The `-t` flag tags the image, and the `-f` flag lets you specify the name and location of the Dockerfile.

Page 159

> ...Docker Compose, which deploys and manages multi-container applications on Docker nodes operating in **single-engine mode**.

> ...Docker Stacks. Stacks deploy and manage multi-container apps on Docker nodes operating in **Swarm mode**

### What is a Dockerfile?

1. Dockerfile is a plaintext doc that contains all the commands a user could call on the command line to assemble an image

2. This is used for creating an automated build that executes several command-line instructions in succession.

### How does docker build command work?

1. It builds an image from the Dockerfile and **context**
    1. #### What is a build context?
        1. The build’s context is the set of files at a specified location PATH or URL.
        2. The PATH is a directory on your local filesystem. 
        3. The URL is a Git repository location
        4. Build context is processed recursively
        5. Path includes all the subdirectories and the URL includes repo and its submodules
    2. #### What happens when the <docker build .> command is given?
        1. The first thing it does is send the entire context ( recursively to the daemon )
        > **Best practice: **
        > * Make sure to do the docker build in an empty directory with only the files necessary for the image to build
        > * If you can't do that, make sure to add the unnecessary files and folders to .dockerignore file
   3. #### What happens when the Docker daemon reads each instruction in Dockerfile?
        1. It creates a new image if necessary before finally outputting the ID of the new image
        2. Each instruction is run independantly. i.e RUN cd /tmp will have no effect if you give COPY in the next line as its a separate image
        3. Whenever possible, Docker used build-cache to accelerate the process
    
### Format
```Dockerfile
#Comment
INSTRUCTION arguments
```
1. FROM
   > * Dockerfile must start with FROM instruction.
   > * Can only have ARG to substitute variables in FROM before it apart from comments
   > * In case of multi build, we can give name by **AS name**
2. LABEL
   > * Adds metadata to image and does not commit a new image 
   > * Its a key value pair
3. MAINTAINER(deprecated)
4. ENV
   > * Key value pair

5. ADD
   > * Allows src to be a remote URL or a tar and auto extract it to dest
6. COPY
   > * For copying files from your local filesystem to destination file in docker image
   > * **Best practices**
   > * Prefer COPY over ADD
   > * COPY accepts a flag called --from=<image_name> that can be used to set the source location to a previous build stage (created with FROM .. AS <name>
7. WORKDIR
    > * The WORKDIR instruction sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile.

7. RUN
   > * Can be exec (json) and shell form
   > * Creates a new layer and commits to a image. This image is passed to the next step
8. ENTRYPOINT 
   > * An ENTRYPOINT allows you to configure a container that will run as an executable.
   > * Dockerfile should specify either one of CMD or ENTRYPOINT commands
9. CMD
   > * The main reason for using CMD is to provide defaults
   > * There can be only one CMD in a Docker file, 
   > * The last CMD only will take effect in case of multiple CMD
   > * Arguments specified by docker run will override the default specified in CMD
   > * It differs from RUN as RUN will commit a image and CMD does not execute anything at build time but specifies the intended command for the run time of the image
10. ARG
    > * The ARG instruction defines a variable that users can pass at build-time to the builder with the docker build command using the --build-arg <varname>=<value> flag
11. USER
    > * The USER instruction sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any RUN, CMD and ENTRYPOINT instructions that follow it in the Dockerfile.

12. EXPOSE
    > * The EXPOSE instruction informs Docker that the container listens on the specified network ports at runtime. 
    > * You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.
    > * Does not actually publish the port. Serves as documentation
    > * Port gets published by docker run -p 80:80 ...
    
13. VOLUME
    > * The VOLUME instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.
    
14. ONBUILD
15. STOPSIGNAL
16. HEALTHCHECK
17. SHELL
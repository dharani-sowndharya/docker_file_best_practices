* A Docker image consists of read-only layers each of which represents a Dockerfile instruction. 
* The layers are stacked and each one is a delta of the changes from the previous layer.
```Dockerfile
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
* Each instruction creates one layer:

> * FROM creates a layer from the ubuntu:18.04 Docker image.
> * COPY adds files from your Docker client’s current directory.
> * RUN builds your application with make.
> * CMD specifies what command to run within the container.
> * When you run an image and generate a container, you add a new writable layer (the “container layer”) on top of the underlying layers. 
> * All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this writable container layer.


1. Try to see if you can reduce the size of the image. Think if a particular layer or dependency is needed before adding it
What am I trying to improve? Faster deployments and reducing attack surface
2. FROM image best practices
    1. Use the official Docker image and make sure that it is a repo that is frequently updated
        Why? It would have followed the best practices and will be more secure
        ```Dockerfile
        # pull official base image
         FROM node:13.12.0-alpine
    
        # set working directory
        WORKDIR /app
    
        # add `/app/node_modules/.bin` to $PATH
        ENV PATH /app/node_modules/.bin:$PATH```
    2. Avoid using latest. Go for specific tags
      Why? Latest might have some breaking changes
        Eg: node:13.12.0 over node:latest
    3. Use flavors of lightweight images like alpine
        Why? Creates smaller footprint and is very lightweight and as a result, its fast
     Eg: docker pull nginx:1.20 and docker pull nginx:1.20-alpine
     docker image ls and see the diff
    4. Build context
        1. "." copies everything in your current directory in docker build .
        2. This is insecure as it might copy unnecessary/sensitive info to the Docker build
        3. Always create a folder and use docker build -t demo demo_files/
        4. Also do not use ADD . /some_folder when copying the entire app code base
           Why? If any single file changes like the log file/test reports in the entire build context, Docker will invalidate the cache and will rebuild it again.
           This will significantly increase the time taken for the build. So add only specfic files. If you have a lot of files to add , utilize the next point
        5. Always have .dockerignore and update it so that the unnecessary files that could be present is not copied to the docker image
        6. Also, having a smaller build context will make your builds faster and efficient
        
   
 
3. Do not use multiple RUN commands as each RUN command will build a layer. Try to install all packages with a single RUN command. 
What gets affected? Performance and efficiency
   
```Dockerfile
RUN apt-get update && apt-get install -y \
   httpd \
   nginx \
   build-essential \
   curl \
   node1.0 \
   libcap-dev \
   libsqlite3-dev \
   git \
   awscli \
   jdk1.8 \
   s3cmd=1.1.*
```

4. Build:
    1. Leverage build cache
       Why? If your build contains different layers, keep the frequently changing layers at the end and the constant layers at the top.
       For example, if you need to install OS packages, place them on the top(This is relatively slow) so that in the subsequent build docker will use the cache from the previous build and you need not wait every single time
   2. Build the source app in a managed environment
       Why? Building in local might create inconsistency as the packages you have installed in your local could have an effect on the build
       Eg: jdk version changes could significantly affect your build
  3.  Multi Stage build. Build the first image using a unique build image and copy the compiled binaries to your actual container image to run
      Why? Solves the issue of managed env and also reduces unnecessary file size that could happen if its a single image
      Eg: cd multi_stage_build ; docker build -t demo -f Dockerfile1 .
  


5. Security:
   1. Do not run containers as root. Might need additonal steps like creating a user, giving permissions to specific folder where the process needs permission
    Why? Least privileges as best practice
    Eg: cd security && docker build -t nonroot -f DockerfileNonRoot .
  
6. Other:
    1. Sort multi line args alphanumerically
    Why? We can make sure that we do not add the same package twice. PRs can be easier to review
   ```Dockerfile
       RUN apt-get update && apt-get install -y \
       bzr \
       cvs \
       git \
       mercurial \
       subversion \
       && rm -rf /var/lib/apt/lists/*   
    ```
7. Utilize --no-cache=true when you want to intentionally invalidate the cache

    Eg: Git clone with old tag
    ```
    RUN git clone https://github.com/bdehamer/dot_files.git WORKDIR /dot_files 
    RUN git checkout v1.0.0
    ```
    Git clone with new tag
    ```
RUN git clone https://github.com/bdehamer/dot_files.git WORKDIR /dot_files 
RUN git checkout v1.1.0
```

If we do Docker build now, it'll throw an error that the tag does not exist. This is because the git clone command is cached. To avoid this we can use
```
docker build -t demo --no-cache=true ./Dockerfile
```
Or change the Dockerfile to
```
RUN git clone https://github.com/bdehamer/dot_files.git WORKDIR /dot_files && git checkout v1.1.0
```


###References:
1. https://sysdig.com/blog/dockerfile-best-practices/
2. https://blog.bitsrc.io/best-practices-for-writing-a-dockerfile-68893706c3
3. https://www.ctl.io/developers/blog/post/more-docker-image-cache-tips
4. https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
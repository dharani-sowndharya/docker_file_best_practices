###List docker images
1. docker image ls | grep nginx

### Expose port of nginx in 80 and run a docker container
docker run -p 80:80 --name demo -d nginx

### Connect to Docker container and add a file
docker exec -it demo /bin/bash
cd /usr/share/nginx/html
echo "This is a demo" > demo.html
exit

### Open in web browser to see the file that you created
http://localhost:80/demo.html

### To create an image/snapshot of the running container
docker commit demo mydemo
docker image ls | grep demo

### Stop and remove the container
docker stop demo
docker rm demo

### Now run the container with the new image we created 
docker run -p 80:80 --name mydemocontainer -d mydemo

### Go to the localhost:80/demo.html and now you can see your custom html there
docker container pull busybox
docker pull busybox

docker scout quickview busybox


docker image ls
docker images


docker image inspect busybox
docker image inspect busybox | grep Hostname

  
docker image history busybox


docker pull amazonlinux
docker images
docker container -- help
docker images ls
docker image ls
docker run --name alinux01 -dt amazonlinux


docker container exec -it alinux01 bash
docker ps
docker container commit alinux01 alinux02
docker container ps
docker container ps -a
docker container images ls
docker container images
docker images
docker container stop alinuxp01
docker container alinux01
docker container stop alinux01
docker images
docker history alinux02
docker container run -dt --name alinux02 alinux02
docker container ps
docker container -it exec alinux02 bash
docker container -it docker-exec alinux02 bash
docker container exec -it alinux02 bash
docker container stop alinux02
docker container ps -a
docker container rm -all
docker system df

# To get all containers list
docker container stop $(docker container ls -aq)
docker container rm $(docker container ls -aq)

docker pull ubuntu

docker rmi 4858f073da7e (image name)

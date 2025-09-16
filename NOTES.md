# Mini-Projet Commun Docker
BootCamp DevOps - Mini Projet Commun - Docker

## ARCHITECTURE OF THE PROJECT'S MICROSERVICES IN THE DOCKER ENGINE

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9e5c8420-9879-4114-8519-5473c69c5581" />

# BUILD AND TEST

<!-- Considering you just have cloned this repository, you have to follow those steps to get the 'student_list' application ready : -->

## 1- Change directory and build the api container image :
   - cd ./mini-projet-commun-docker/simple_api
   - docker build . -t pozos-api:v1
   - docker images

## 2- Create a bridge-type network for the two containers to be able to contact each other by their names thanks to dns functions :
   - docker network create pozos_network --driver=bridge
   - docker network ls

## 3- Move back to the root dir of the project and run the backend api container with those arguments :
   - cd ..
   - docker run --rm -d --name pozos_api --network pozos_network -v ./simple_api/:/data/ pozos-api:v1
   - docker ps

<!-- As you can see, the api backend container is listening to the 5000 port. This internal port can be reached by another container from the same network so I chose not to expose it. -->

<!-- I also had to mount the ./simple_api/ local directory in the /data/ internal container directory so the api can use the student_age.json list -->

## 4- Update the index.php file :

   - You need to update the following line before running the website container to make api_ip_or_name and port fit your deployment  $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';

   - Thanks to our bridge-type network's dns functions, we can easyly use the api container name with the port we saw just before to adapt our website

     sed -i 's|<api_ip_or_name:port>|pozos_api:5000|g' ./website/index.php

## 5- Run the frontend webapp container :
   Username and password are provided in the source code simple_api/student_age.py

   - docker run --rm -d --name pozos_webapp -p 80:80 --network pozos_network -v ./website/:/var/www/html -e USERNAME=toto -e PASSWORD=python php:apache
   - docker ps

## 6- Test the api through the frontend :

   Using command line :

   The next command will ask the frontend container to request the backend api and show you the output back. The goal is to test both if the api works and if frontend can get the student list from it.

   - docker exec pozos_webapp curl -u toto:python -X GET http://pozos_api:5000/pozos/api/v1.0/get_student_ages

   Using a web browser IP:80 

   If you're running the app into a remote server or a virtual machine (e.g provisionned by eazytraining's vagrant file), please find your ip address typing hostname -I

  - If you are working on PlayWithDocker, just open the 80 port on the gui
  - If not, type localhost:80

## 7- Clean the workspace :
   Thanks to the --rm argument we used while starting our containers, they will be removed as they stop. Remove the network previously created.

  - docker stop pozos_api
  - docker stop pozos_webapp
  - docker network rm pozos_network
  - docker network ls
  - docker ps

--------------------------------------------------

# DEPLOYMENT

  <!-- As the tests passed we can now 'composerize' our infrastructure by putting the docker run parameters in infrastructure as code format into a docker-compose.yml file. -->

  ## 1- Run the application (api + webapp) :
     As we've already created the application image, now you just have to run :

    - docker-compose up -d

    Docker-compose permits to chose which container must start first. The api container will be first as I specified that the webapp depends_on: it.

  ## 2- Create a registry and its frontend
     I used registry:2 image for the registry, and joxit/docker-registry-ui:static for its frontend gui and passed some environment variables

     E.g we'll be able to delete images from the registry via the gui.

     - docker-compose -f docker-compose-registry.yml up -d
     - on browser type: ip_add:8080

## 3- Push an image on the registry and test the gui
   You have to rename it before (:latest is optional) :

   NB: for this exercise, I have left the credentials in the .yml file.

   - docker login localhost:5000
<img width="675" height="162" alt="image" src="https://github.com/user-attachments/assets/678f0d29-307f-4dee-9070-cc68b5a4466d" />
   
   - docker image tag pozos-api:v1 localhost:5000/pozos/pozos-api:v1   
   - docker images
<img width="705" height="182" alt="image" src="https://github.com/user-attachments/assets/daf4e22b-067d-4399-94ad-b553875ad8d1" />
   
   - docker image push localhost:5000/pozos/pozos-api:v1
<img width="764" height="199" alt="image" src="https://github.com/user-attachments/assets/3ea433c3-3198-492d-958b-eb0bacb33e0c" />

  reload the browser to see the pushed image  
<img width="947" height="413" alt="image" src="https://github.com/user-attachments/assets/d9782746-c972-47be-aa37-9d8f594e368a" />
##
<img width="950" height="410" alt="image" src="https://github.com/user-attachments/assets/9194f0b6-8edd-4a12-8b10-4869d4cba149" />


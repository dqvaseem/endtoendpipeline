------    Completion steps →
upload the source code to github
setup circle ci and connect the github repository
upload the .circleci/config.yml file to build pipeline
This ci -cd pipeline build your docker-image and sent it to dockerhub
setup ec2 on aws and take ssh of it
setup minikube and create a sample pod.yml file
start your kubernetes cluster
Step 1 → Upload code to github
Here we are not going to see how we setup if you have any issue in pushing your code to github you could checkout my previous blogs in git in my profile

check out this blog of mine for github error

first you need to clone the following repository
git clone https://github.com/Aakibgithuber/Devops-end-to-end-deployment

2. Now push the exact to your github account and move forward to Circle ci step

Step 2 → setup circle ci and connect with github repository
Sign into your circle ci account and if you don’t have an account just sign up by the following steps

go to circle ci sign up page
enter your email and create your password

3. Choose the following options


4. Now you have to choose github and you redirect to your github accout select the repository where your code present and then click on authorize


5. now you have tocreate a project for which you have to require a key pairs


go to your teminal and generate a key pairs in which public key is paste to deploy key options in github and private key on your create project page
run → ssh-keygen
cd /root/.ssh
you will find id_rsa(private key) and id_rsa.pub(public key) just copy paste the on the location mentioned above
Now you are connected to your github repository

Step 3 →upload the .circleci/config.yml file to build pipeline
copy the following code and make a folder .cricleci/config.yaml and paste into it and push it to your github repo
version: 2.1
executors:
  docker-publisher:
    environment:
      IMAGE_NAME: aakibkhan1212/building-on-ci
    docker:
      - image: circleci/buildpack-deps:stretch
jobs:
  build:
    executor: docker-publisher
    steps:
      - checkout
      - setup_remote_docker
      - run:
          name: Build Docker image
          command: |
            docker build -t $IMAGE_NAME:latest .
      - run:
          name: Archive Docker image
          command: docker save -o image.tar $IMAGE_NAME
      - persist_to_workspace:
          root: .
          paths:
            - ./image.tar
  publish-latest:
    executor: docker-publisher
    steps:
      - attach_workspace:
          at: /tmp/workspace
      - setup_remote_docker
      - run:
          name: Load archived Docker image
          command: docker load -i /tmp/workspace/image.tar
      - run:
          name: Publish Docker Image to Docker Hub
          command: |
            echo "$DOCKERHUB_PASSWWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
            docker push $IMAGE_NAME:latest
workflows:
  version: 2
  build-master:
    jobs:
      - build:
          filters:
            branches:
              only: master
      - publish-latest:
          requires:
            - build
          filters:
            branches:
              only: master
2. Add the environment variable by click on project setting option you will find Environment varible option

$DOCKERHUB_USERNAME →your dockerhub username

$DOCKERHUB_USERNAME → your dockerhub password

now it will automatically starts building your docker image and push it to your dockerhub account


Step 4 setup ec2 on aws and take ssh of it
go to your aws account and launch ubuntu machine and take ssh it into your terminal
commands to run on your terminal

a. ssh -i <your key pair name > ec2-user@<public ip>

c. apt update

Step 5 setup Minikube for kubernetes
command to run on your terminal

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start — driver docker
kubectl get po -A
Now we have to write the sample.yaml file for pod deployment →
Below is the example file just replace the image name that is present in dockerhub repo


once you created a file then you have to run a command to craete a pod from file

kubectl apply -f pod sample.yaml
kubectl get po -o wide
you will see your pod is created and running having a private ip which you could use in checking

minikube ssh
curl -L http://<your pod ip>:<your port no.>
you will see your aplication is running inside a minikube cluster change now you are free to make changes in yaml file such as creating replicas etc.

Thanks for reading ….

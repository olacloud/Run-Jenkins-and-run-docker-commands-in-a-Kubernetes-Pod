Run Jenkins and run docker commands in a Kubernetes Pod
Run Jenkins in kubernetes and in order to execute Docker commands inside Jenkins nodes, we run the docker:dind Docker image inside the same pod.

1. Create a namespace called jenkins
2. Create a persistent storage with a PVC(Persistent Volume Claim)
2. Create a Pod with 2 containers in this jenkins namespace
The first container is a customized official Jenkins Docker image derived from a Dockerfile with the following content and pushed to docker

FROM jenkins/jenkins:2.426.1-jdk17
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"

The second container is the docker:dind image which allows you to run docker commands through a Docker daemon port to control this inner Docker daemon.

3. Create a NodePort service to access Jenkins on port 30000 (optional the jenkins JNLP port)

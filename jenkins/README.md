# Dokerfile

```DOKERFILE
FROM jenkins/jenkins:2.452.2-jdk17

USER root

RUN apt-get update && apt-get install -y lsb-release

RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \

https://download.docker.com/linux/debian/gpg

RUN echo "deb [arch=$(dpkg --print-architecture) \

signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \

https://download.docker.com/linux/debian \

$(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list

RUN apt-get update && apt-get install -y docker-ce-cli

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

RUN groupadd -f docker && usermod -aG docker jenkins

RUN chmod 666 /var/run/docker.sock

USER jenkins

RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```

# Création Network

```bashh
docker network create jenkins
```

# Build image Docker

```bash
docker build -t myjenkins-blueocean:2.452.2-1 .
```

# Run

```
 docker run --name jenkins-blueocean --restart=on-failure --detach \
 --network jenkins --env DOCKER_HOST=unix:///var/run/docker.sock \
 --volume jenkins-data:/var/jenkins_home \
 --volume /var/run/docker.sock:/var/run/docker.sock \
 --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.452.2-1
```

# Plugins Jenkins et Tools

### plugins :

- Docker Pipeline
- GitHub Integration
- Maven Integration

### tools :

Maven : - Nom : Maven-3.9.8 ( même nom que dans Jenksfile) - installation automatique
Credentials :

- Pour Docker il faut un token à la place du MDP

# PROJET

## Dockerfile

```

FROM openjdk:11-jre-slim

WORKDIR /app

COPY target/jenkins-first-project-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Jenkinsfile

```
pipeline {
    agent any
    environment {
	    DOCKER_USERNAME =  'ncherfaoui'
	    GITHUB_REPO_URL =  'https://github.com/NCherfaoui/jenkins-first-project.git'  }

    tools {
        maven 'Maven-3.9.8'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${env.GITHUB_REPO_URL}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_USERNAME}/calculatrice:${env.BUILD_NUMBER}", '.')
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${env.DOCKER_USERNAME}/calculatrice:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
    }
}
```

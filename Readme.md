# Microservices Config Server

This project is a Spring Cloud Config Server that serves as the central configuration hub for a microservices architecture. It is part of a larger microservices ecosystem that utilizes various Spring Cloud tools for distributed systems, such as Eureka for service discovery and Spring Boot Admin for monitoring.

## Features

- Centralized configuration management using Spring Cloud Config Server.
- Integration with Eureka for service discovery.
- Monitoring with Spring Boot Admin.
- Docker support for easy deployment.
- Jenkins pipeline for automating build, test, and deployment.

## Prerequisites

To run this project locally, ensure you have the following installed:

- Java 17 or later
- Maven 3.9.5 or later
- Docker (for building and running containerized services)
- Git (for cloning the repository)
- Jenkins (for setting up CI/CD pipelines)

## Technologies Used

- Spring Boot
- Spring Cloud Config
- Spring Cloud Netflix Eureka
- Spring Boot Admin
- Docker
- Jenkins

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/bhupendrasambare/microservices-config-server.git
cd microservices-config-server
```

### Build the Project

```bash
mvn clean package -DskipTests
```

### Running the Application

You can run the Config Server locally using Maven:

```bash
mvn spring-boot:run
```

The Config Server will start on port `8888` by default.

### Dockerizing the Application

#### Dockerfile

The following `Dockerfile` is used to create a Docker image for the Config Server:

```dockerfile
# Start with a base image containing Java runtime
FROM openjdk:17-jdk-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the packaged JAR file into the container at /app
COPY target/*.jar app.jar

# Expose the port that the application listens to
EXPOSE 9003

# Specify the command to run your application
CMD ["java", "-jar", "app.jar", "--custom.server-ip=${CUSTOM_SERVER_IP}"]
```

#### Build and Run the Docker Image

To build and run the Docker container for the Config Server, follow these steps:

1. Build the Docker image:

```bash
docker build -t bhupendra1404/microservice:ms-config .
```

2. Run the Docker container:

```bash
docker run -d -p 9005:9005 --name ms-config -e CUSTOM_SERVER_IP=192.168.29.226 bhupendra1404/microservice:ms-config
```

### Jenkins Pipeline

The project includes a Jenkins pipeline for automating the build, test, and deployment processes. The pipeline performs the following steps:

- Clones the repository from GitHub.
- Builds the project and packages it into a JAR.
- Builds the Docker image using the packaged JAR.
- Deploys the Docker container.
- Cleans up dangling Docker images after deployment.

#### Jenkins Pipeline Configuration

```groovy
pipeline {
    agent any

    environment {
        JAVA_HOME = "/Users/bhupendrasam1404/Library/Java/JavaVirtualMachines/jdk-22.0.1.jdk/Contents/Home"
        DOCKER_IMAGE = "bhupendra1404/microservice:ms-config"
        CONTAINER_NAME = "ms-config"
        DOCKER_PATH = '/usr/local/bin/docker'
        MAVEN_PATH = '/opt/homebrew/Cellar/maven/3.9.5/libexec/bin/mvn'
        CUSTOM_SERVER_IP = '192.168.29.226'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bhupendrasambare/microservices-config-server.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh "${MAVEN_PATH} clean package -DskipTests -Dcustom.server-ip=192.168.29.226"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "${DOCKER_PATH} build --no-cache -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Deploy Docker Image') {
            steps {
                script {
                    sh """
                    if [ \$(${DOCKER_PATH} ps -a -q -f name=${CONTAINER_NAME}) ]; then
                        ${DOCKER_PATH} stop ${CONTAINER_NAME}
                        ${DOCKER_PATH} rm ${CONTAINER_NAME}
                    fi
                    """

                    sh """
                    ${DOCKER_PATH} run -i -p 9005:9005 -d --name ${CONTAINER_NAME} -e CUSTOM_SERVER_IP=${CUSTOM_SERVER_IP} ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Clean Up Docker Images') {
            steps {
                script {
                    // Remove all dangling images
                    sh "${DOCKER_PATH} image prune -f"
                }
            }
        }
    }
}
```

### Running the Jenkins Pipeline

Once Jenkins is installed and configured, create a pipeline job, and copy the above pipeline script into the job's configuration. Trigger the pipeline to build, deploy, and manage the Docker container automatically.

### Accessing the Application

- Config Server: `http://localhost:9005`

## Configuration Management

To use this Config Server, configure other services in your microservices architecture by pointing them to this server in their application properties or YAML file:

```yaml
spring:
  cloud:
    config:
      uri: http://localhost:9005
```

## License

This project is licensed under the MIT License.

## Contact

For more information, you can contact the repository owner at [bhupendrasam1404@gmail.com](mailto:bhupendrasam1404@gmail.com).
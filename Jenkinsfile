// this Jenkins pipeline is for Eureka deployment

pipeline {
    agent {
        label 'k8s-slave'
    }

    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }

    environment {
        APPLICATION_NAME = "eureka" 
        SONAR_TOKEN =  credentials('sonar-creds2')
        SONAR_URL = "http://34.60.91.201:9000"
        // if any errors with readMavenPom, make sure pipeline-utility-steps plugin is install in your jenkins, if not do install
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/vishnu7674"
        DOCKERHUB_CREDS = credentials('dockerhub_creds')

    }
    stages {
        stage ('Build') {
            steps {
                script{
                    sh """
                    echo "Building the ${env.APPLICATION_NAME} Application"
                    mvn clean package -DSkipTests=true
                """

                }
                
            }
        }
        stage ('sonar') {
            steps {
                script {
                    echo "Starting sonar scan"
                    withSonarQubeEnv('SonarQube'){  //the name we saved in system under manage jenkins
                        sh """
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=i27-eureka \
                            -Dsonar.host.url=${env.SONAR_URL} \
                            -Dsonar.login=${SONAR_TOKEN}
                    """
                }
                timeout (time: 2, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }

                }
            }
        }
        stage ('Docker build and push') {
            steps {
                // existing artifact format: i27-eureka-0.0.1-SNAPSHOT.jar
                // My Destination artificat format: i27-eureka-buildnumber-branchname.jar
                echo "My JAR Source: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                echo "MY JAR Destination: i27-${env.APPLICATION_NAME}-${BUILD_NUMBER}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                sh """
                    echo "****************************** Building docker image *************************************"
                    pwd
                    ls -la 
                    cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
                    ls -la ./.cicd
                    docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd
                    #docker.io/vishnu7674/eureka:
                    echo "******************** Login to docker registry ******************************************"
                    docker login -u ${DOCKERHUB_CREDS_USR} -p ${DOCKERHUB_CREDS_PSW}
                    docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}
                """
            }
        }
        stage ('Deploy to dev') {
            steps {
                echo "Deploying to dev server"
                withCredentials([usernamePassword(credentialsId: 'maha_ssh_docker_server_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {

                    script {
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip \"docker pull ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} \""
                        try {
                            // stop the container
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip docker stop ${env.APPLICATION_NAME}"
                            // remove the continer
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip docker rm ${env.APPLICATION_NAME}"
                        }
                        catch(err) {
                            echo "Error caught: $err"
                        }
                        // create the container
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$dev_ip docker run -dit --name ${env.APPLICATION_NAME}-dev -p 5761:8761 ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}" 
                    }
                }
            }
        }
    }
}
// create a container
                // docker container create imagename
                // docker run -dit --name containername imageName
                // docker run -dit --name eureka-dev
               // docker run -dit --name ${env.APPLICATION_NAME}-dev -p 5761:8761 ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} 
               // run -dit --name ${env.APPLICATION_NAME}-dev -p 5761:8761
//sshpass -p password ssh -o StrictHostKeyChecking=no username@dockerserverip
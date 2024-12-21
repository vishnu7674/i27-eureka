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
        stage ('Docker') {
            steps {
                echo 'currently in docker stage'
            }
        }
    }
}
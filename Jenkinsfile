// This Jenkinsfile is for Eureka Deployment

pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        maven 'Maven-3.9.11'
        jdk 'JDK-17'
    }
    environment {
        APPLICATION_NAME = "Eureka"
        SONAR_TOKEN = credentials('sonar_creds')
        SONAR_URL = "http://34.46.97.238:9000"
    }
    //Stages
    stages {
        stage ('Build') {
            steps {
                echo "Building the ${env.APPLICATION_NAME} Application"
                sh 'mvn clean package -DskipTests=true'
                // After building if you face any issue with mvn installation? --> There is issue with the permissions (chown -R <kommuamar1133>:<kommuamar1133> <apache maven) of maven in /opt/apcahe_maven
            }
        }
        stage ('Sonar') {
            steps {
                echo "Starting Sonar Scan"
                withSonarQubeEnv('SonarQube'){  // The name you saved in system under manage jenkins
                    sh """
                    mvn sonar:sonar \
                        -Dsonar.projectkey=i27-eureka \
                        -Dsonar.host.url=${env.SONAR_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }  
                timeout (time: 2, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Docker') {
            steps {
                echo "Currently in Docker stage"
            }
        }
    }
}
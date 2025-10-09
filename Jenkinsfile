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
                sh """
                mvn sonar:sonar \
                    -Dsonar.projectkey=i27-eureka \
                    -Dsonar.host.url=http://34.46.97.238:9000/ \
                    -Dsonar.login=squ_ecd1a5d6513762c30f73a9938c1a41823b88a49d
                """
            }
        }
    }
}
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
        APPLICATION_NAME = "eureka"
        SONAR_TOKEN = credentials('sonar_creds')
        SONAR_URL = "http://34.46.97.238:9000"
        // https://www.jenkins.io/doc/pipeline/steps/pipeline-utility-steps/#readmavenpom-read-a-maven-project-file
        // If any errors with readMavenPom, make sure pipeline-utility-steps plugin is installed in your jenkins, if not do install it
        // Script Approval issues : http://34.148.12.185:8080/scriptApproval/
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/kommuamar1133"

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
                // Existing artifact format: i27-eureka-0.0.1-SNAPSHOT.jar
                // My Destination artifact format: i27-eureka-buildnumber-branchname.jar
                echo "My JAR file SOURCE: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                echo "My JAR Destination: i27-${env.APPLICATION_NAME}-${BUILD_NUMBER}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                sh """
                    echo "***********************Building Docker Image*************************"
                    pwd
                    ls -la
                    # docker build --no-cache -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} .cicd/opt/i27
                    # docker build -t imagename dockerfilepath
                """
            }
        }
    }
}
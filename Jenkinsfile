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
        DOCKER_CREDS = credentials('dockerhub_creds')
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
        stage ('Docker Build & Push') {
            steps {
                // Existing artifact format: i27-eureka-0.0.1-SNAPSHOT.jar
                // My Destination artifact format: i27-eureka-buildnumber-branchname.jar
                echo "My JAR file SOURCE: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                echo "My JAR Destination: i27-${env.APPLICATION_NAME}-${BUILD_NUMBER}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                sh """
                    echo "***********************Building Docker Image*************************"
                    pwd
                    ls -la
                    cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}  ./.cicd
                    ls -la ./.cicd
                    docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd
                    # docker build -t imagename dockerfilepath
                    echo "***********************Login to Docker Registry*******************************"
                    # docker login -u username -p password
                    docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                    # docker push image_name
                    docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}

                """
            }

        }
        stage ('Deploy to Dev') {
            steps {
                echo "Deploy to Dev"
                withCredentials([usernamePassword(credentialsId: 'navya_ssh_dockerserver_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    //sshpass -p password ssh -o StrictHostKeyChecking=no username@dockerserver_ip
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@dev_ip \"hostname -i\""
                }
            }
            
        }
    }
}

//We need to connect to the dockerserver through the jenkinsslave using below command:
//withCredentials([usernameColonPassword(credentialsId: 'mylogin', variable: 'USERPASS')]) {
  // sshpass -p password ssh -o StrictHostKeyChecking=no username@dockerserver_ip



// usernameVariable : String
// Name of an environment variable to be set to the username during the build.
// passwordVariable : String
// Name of an environment variable to be set to the password during the build.
// credentialsId : String
// Credentials of an appropriate type to be set to the variable.
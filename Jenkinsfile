pipeline {
    agent any

    environment {
        EC2_IP = '13.201.26.185'
        EC2_USER = 'ubuntu'
        TOMCAT_WEBAPPS = '/var/lib/tomcat10/webapps'
        SONAR_PROJECT_KEY = 'petclinic'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/navaneethan0312/petclinic_war.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {  // Must match SonarQube server name in Jenkins config
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName="Pet Clinic"
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-credentials']) {  // Must match your credentials ID
                    sh '''
                        # Copy WAR to EC2
                        scp -o StrictHostKeyChecking=no \
                            target/*.war \
                            ${EC2_USER}@${EC2_IP}:/tmp/petclinic.war

                        # Move WAR into Tomcat webapps and restart
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} "
                            sudo cp /tmp/petclinic.war ${TOMCAT_WEBAPPS}/petclinic.war &&
                            sudo systemctl restart tomcat10
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed! App deployed to http://${EC2_IP}/petclinic'
        }
        failure {
            echo 'Pipeline failed. Check the logs above.'
        }
    }
}

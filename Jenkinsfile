pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        SERVER_USER = 'ubuntu'
        SERVER_IP   = '16.171.141.170'
        SERVER_PATH = '/home/ubuntu/smart'
        APP_NAME    = 'S-Mart-0.0.1-SNAPSHOT.jar'
        SSH_KEY     = '/var/lib/jenkins/.ssh/devops.pem'
        SSH_OPTS    = '-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Kadambari-Ozarkar/S-Mart.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                sh """
                # 1️⃣ Create folder on remote server
                ssh ${SSH_OPTS} -i ${SSH_KEY} ${SERVER_USER}@${SERVER_IP} "mkdir -p ${SERVER_PATH}" || true

                # 2️⃣ Copy the JAR to EC2
                scp ${SSH_OPTS} -i ${SSH_KEY} target/${APP_NAME} ${SERVER_USER}@${SERVER_IP}:${SERVER_PATH}/ || true

                # 3️⃣ Stop old app (if running) and start new app
                ssh ${SSH_OPTS} -i ${SSH_KEY} ${SERVER_USER}@${SERVER_IP} "pkill -f S-Mart || true; nohup java -Xmx512m -jar ${SERVER_PATH}/${APP_NAME} > ${SERVER_PATH}/app.log 2>&1 &" || true
                """
            }
        }

    }

    post {
        success {
            echo '✅ Deployment Completed Successfully'
        }
        failure {
            echo '❌ Deployment Failed'
        }
    }
}

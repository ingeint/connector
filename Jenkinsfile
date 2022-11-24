pipeline {
    agent none
    stages {
        stage('Clone and Copy module ${PROJECT_NAME} dev') {
            agent {
                label 'sosnet_1'
            }
            environment {
                CONTAINER = sh(returnStdout: true, script: 'docker ps --filter "name=$CONTAINER_NAME" --format "{{.ID}}"').trim()
                ADDON_PATH = "/mnt/extra-addons/${PROJECT_NAME}"
                ADDON_FOLDER = "/tmp/addons"
            }
            steps {
                sh 'cp /var/jenkins_home/.ssh/id_rsa* /root/.ssh/'
                sh 'docker exec -u root $CONTAINER rm -rf $ADDON_PATH'
                sh 'git clone $PROJECT_URL -b $BRANCH'
                sh 'docker cp ${PROJECT_NAME} $CONTAINER:$ADDON_PATH'
                sh 'rm -R ${PROJECT_NAME}'
            }
        }
        stage('Update DB') {
            agent {
                docker {
                    image 'postgres:13'
                    label 'sosnet_1'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'odoo-creds', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'psql -h "$DB_SERVER" -p 5432 -U $user -d "$DB_NAME" -c "update ir_module_module set state = \'to upgrade\' where name in $ACTIVE_MODULES"'
                }
            }
        }
        stage('Restart Container'){
            agent {
                label 'sosnet_1'
            }
            environment {
                CONTAINER = sh(returnStdout: true, script: 'docker ps --filter "name=$CONTAINER_NAME" --format "{{.ID}}"').trim()
            }
            steps {
                sh 'docker rm -f $CONTAINER'
            }
        }
    }
}
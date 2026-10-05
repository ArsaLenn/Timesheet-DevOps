pipeline {
    agent any

    environment {
        IMAGE_NAME      = "backend-app"
        REGISTRY_USER   = "arsaaa"
        REGISTRY_IMAGE  = "arsaaa/backend-app:latest"
        NETWORK_NAME    = "timesheet-net"
        MYSQL_URL       = "jdbc:mysql://mysql:3306/timesheet-devops-db?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ArsaLenn/Timesheet-DevOps.git'
            }
        }

        stage('Build & Test Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t backend-app:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker tag backend-app:latest $REGISTRY_IMAGE'
                    sh 'docker push $REGISTRY_IMAGE'
                }
            }
        }

        stage('Déploiement MySQL') {
            steps {
                sh 'docker network create $NETWORK_NAME || true'
                sh 'docker rm -f mysql || true'
                sh '''
                    docker run -d --name mysql \
                        --network $NETWORK_NAME \
                        -e MYSQL_ALLOW_EMPTY_PASSWORD=yes \
                        -e MYSQL_DATABASE=timesheet-devops-db \
                        -p 3306:3306 \
                        mysql:8.0
                '''
                sh '''
                    echo "Attente du demarrage de MySQL..."
                    for i in $(seq 1 30); do
                        if docker exec mysql mysqladmin ping -h localhost --silent; then
                            echo "MySQL est pret"
                            break
                        fi
                        sleep 2
                    done
                '''
            }
        }

        stage('Déploiement backend-app') {
            steps {
                sh 'docker stop backend-app || true'
                sh 'docker rm backend-app || true'
                sh 'docker pull $REGISTRY_IMAGE'
                sh '''
                    docker run -d --name backend-app \
                        --network $NETWORK_NAME \
                        -e SPRING_DATASOURCE_URL="$MYSQL_URL" \
                        -e SPRING_DATASOURCE_USERNAME=root \
                        -e LOGGING_FILE_NAME=/app/logs/timesheet-devops.log \
                        -p 8082:8082 \
                        $REGISTRY_IMAGE
                '''
            }
        }

        stage('Vérification') {
            steps {
                sh 'sleep 15'
                sh 'docker ps'
                sh 'docker logs backend-app --tail 80'
            }
        }
    }
}

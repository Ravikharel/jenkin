pipeline { 
    agent any
    environment {
        HARBOR_URL = "harbor.registry.local/jenkins1/"
        IMAGE_NAME_FRONTEND = "frontend"
        IMAGE_NAME_BACKEND = "backend"
        IMAGE_NAME_MYSQL = "mysql"
        IMAGE_TAG = "v1"
        HARBOR_PASSWORD = "Harbor12345"
    }

    stages { 
        stage('Building the Images') {
            steps { 
                script { 
                    sh '''
                        docker image build -t ${HARBOR_URL}${IMAGE_NAME_FRONTEND}:${IMAGE_TAG} -f frontend/Dockerfile frontend/
                        docker image build -t ${HARBOR_URL}${IMAGE_NAME_BACKEND}:${IMAGE_TAG} -f backend/Dockerfile backend/
                    '''
                }
            }
        }

        stage('Pushing the Images to Harbor Registry') { 
            steps { 
                script { 
                    sh '''
                        echo ${HARBOR_PASSWORD} | docker login ${HARBOR_URL} -u admin --password-stdin
                        docker push ${HARBOR_URL}${IMAGE_NAME_FRONTEND}:${IMAGE_TAG}
                        docker push ${HARBOR_URL}${IMAGE_NAME_BACKEND}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Pulling Images and Running MySQL Container on Agent Node') { 
            agent { 
                label "vagrant-slave"
            }
            steps { 
                script { 
                    sh '''
                        echo ${HARBOR_PASSWORD} | docker login ${HARBOR_URL} -u admin --password-stdin
                        docker pull ${HARBOR_URL}${IMAGE_NAME_FRONTEND}:${IMAGE_TAG}
                        docker pull ${HARBOR_URL}${IMAGE_NAME_BACKEND}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Creating Docker Network') { 
            agent {
                label "vagrant-slave"
            }
            steps { 
                script { 
                    sh """
                        docker network create naya-network || true
                    """
                }
            }
        }

        stage('Running Docker Containers') { 
            agent {
                label "vagrant-slave"
            }
            steps { 
                script { 
                    sh """
                        docker-compose up -d
                    """
                }
            }
        }

        stage('Waiting for MySQL to be ready') { 
            agent {
                label "vagrant-slave"
            }
            steps {
                script {
                    // Waiting for MySQL to be ready
                    sh '''
                    echo "Waiting for MySQL to be ready..."
                    for i in {1..30}; do
                        if docker exec mysql mysqladmin ping -h localhost -u root -proot --silent; then
                            echo "MySQL is up and running!"
                            break
                        fi
                        echo "Waiting for MySQL to start..."
                        sleep 5
                    done
                    '''
                }
            }
        }

        stage('Configuring MySQL'){ 
            agent {
                label "vagrant-slave"
            }
            steps{
                script{ 
                    sh '''
                        for i in {1..5}; do
                            if docker exec mysql mysql -u root -proot -h localhost -P 3306 -e "USE myapp; CREATE TABLE IF NOT EXISTS users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL);"; then
                                echo "MySQL configuration successful!"
                                break
                            fi
                            echo "Retrying MySQL configuration..."
                            sleep 5
                        done
                    '''
                }
            }
        }
    }
}
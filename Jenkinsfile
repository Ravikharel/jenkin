pipeline { 
    agent{ 
        label "vagrant-slave"
    }
    environment {
        HARBOR_URL = "harbor.registry.local/jenkins1/"
        IMAGE_NAME_FRONTEND = "Nginx"
        IMAGE_NAME_BACKEND = "nodejs"
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


        stage('Running Docker Containers') {
            steps { 
                script { 
                    sh """
                        docker compose up -d 
                    """
                }
            }
        }

        stage('Waiting for MySQL to be ready') { 
            steps {
                script {
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

    }
}
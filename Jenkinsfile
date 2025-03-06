pipeline { 
    agent any
    environment {
        HARBOR_URL = "harbor.registry.local/jenkins1/"
        IMAGE_NAME_FRONTEND = "frontend"
        IMAGE_NAME_BACKEND = "backend"
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

        stage('Pulling Images on Agent Node') { 
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

        stage('Running Docker Containers') { 
            agent {
                label "vagrant-slave"
            }
            steps { 
                script { 
                    sh """
                        
                        docker network create naya-network || true
                        
                        # Stop and remove existing containers if running
                        docker stop mysql frontend backend || true
                        docker rm mysql frontend backend || true

                        
                        docker container run -d -p 3306:3306 --name mysql \
                            -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=myapp \
                            --network naya-network mysql:latest

                        
                        docker container run -d -p 80:80 --name frontend \
                            --network naya-network ${HARBOR_URL}${IMAGE_NAME_FRONTEND}:${IMAGE_TAG}
                        
                        docker container run -d -p 5000:5000 --name backend \
                            --network naya-network ${HARBOR_URL}${IMAGE_NAME_BACKEND}:${IMAGE_TAG}
                    """
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
                        echo "Waiting for MySQL to be ready..."
                        until docker exec mysql mysqladmin ping -h mysql --silent; do
                            sleep 2
                            echo "Waiting for MySQL..."
                        done
                        echo "MySQL is ready. Running SQL commands."
                        docker exec mysql mysql -u root -proot -h mysql -P 3306 -e "
                            USE myapp; 
                            CREATE TABLE IF NOT EXISTS users (
                                id INT AUTO_INCREMENT PRIMARY KEY, 
                                name VARCHAR(255) NOT NULL, 
                                email VARCHAR(255) NOT NULL
                            );
                        "
                    '''
                }
            }
        }
    }
}

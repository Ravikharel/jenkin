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

        stage('Running Docker Container ') { 
            agent {
                label "vagrant-slave"
            }
            steps { 
                script { 
                    sh """
                        
                        docker container run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=myapp --network naya-network mysql:latest
                        docker container run -d -p 80:80 --name frontend --network naya-network ${HARBOR_URL}${IMAGE_NAME_FRONTEND}:${IMAGE_TAG}
                        docker container run -d -p 5000:5000 --name backend --network naya-network ${HARBOR_URL}${IMAGE_NAME_BACKEND}:${IMAGE_TAG}
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
                         docker exec  mysql mysql -u root -proot -h 127.0.0.1 -P 3306 -e "USE myapp; CREATE TABLE IF NOT EXISTS users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL);"

                    '''
                }
            }
        }
    }
}

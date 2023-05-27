pipeline {


 agent any


        environment {

                    NEXUS_USERNAME = 'admin'
                    NEXUS_PASSWORD ='Adnen**91'
                    DOCKER_HUB_USERNAME= 'omardevops'
                    DOCKER_HUB_PASSWORD= 'dckr_pat_MemPoIqVYBryEGwEJXeJMXvLF-M'
                    DOCKER_HUB_SPRING_REPO= 'spring-boot-omar-repo'
                    MYSQL_ROOT_PASSWORD = 'root'
                    MYSQL_PASSWORD = 'root'
                    MYSQL_DATABASE = 'tpachato'


        }

        stages {



        stage("Launch SonarQube and Nexus containers") {
            steps {
                sh ' sudo docker compose -f  /home/vagrant/SonarAndNexus/docker-compose.yml start'
                sh 'echo "date and time BEFORE  sleeping for two minutes" '
                sh 'date +"%m_%d_%Y_%M:%S"'
                sleep(time: 2, unit: "MINUTES")

            }
        }


            stage ('GIT Checkout for Spring Boot App') {
                    steps {
                        echo '...Pulling...';
                        git branch: 'main',
                        url : 'https://github.com/omarsaafi/omarspringbootdevops.git'
                    }
            }


            stage ('Build') {
                    steps {
                       sh "mvn -B -DskipTests clean package"
                    }
            }

            stage ('Test') {
                    steps {
                        sh "mvn test"
                    }
            }

            stage ('Package') {
                    steps {
                        sh "mvn package"
                    }
            }




            stage("build & SonarQube analysis") {
                    steps {
                        sh  "mvn sonar:sonar -Dsonar.projectKey=tpAchatProject -Dsonar.host.url=http://72.10.0.140:9000 -Dsonar.login=8141a4e2160b1ac55882ad543046c4eb0ff436ce"
                    }
            }



            stage('Build Spring Boot image') {

                    steps {
                    sh 'declare BUILD_TAG=$(date +%s-%A-%B)'
                    sh 'docker build -t tp-achat-project:${BUILD_TAG} .'

                    }
            }


            stage('Push Spring Boot image to Nexus') {
                steps {
                        sh 'docker tag tp-achat-project:${BUILD_TAG} 72.10.0.140:8082/docker-devops-repo/tp-achat-project:${BUILD_TAG}'
                        sh 'docker login -u $NEXUS_USERNAME -p $NEXUS_PASSWORD 72.10.0.140:8082'
                        sh 'docker push 72.10.0.140:8082/docker-devops-repo/tp-achat-project:${BUILD_TAG}'
                }
            }


            stage('Push Spring Boot image to Docker Hub') {
                steps {
                        sh 'docker tag tp-achat-project:${BUILD_TAG} docker.io/${DOCKER_HUB_USERNAME}/${DOCKER_HUB_SPRING_REPO}:${BUILD_TAG} '
                        sh 'docker login -u $DOCKER_HUB_USERNAME -p $DOCKER_HUB_PASSWORD'
                        sh 'docker push omardevops/spring-boot-omar-repo:${BUILD_TAG}'

                }
            }


            stage('create Spring boot and Mysql App') {
              steps {

                  sh 'sudo  DOCKER_HUB_USERNAME=${DOCKER_HUB_USERNAME}   DOCKER_HUB_SPRING_REPO=${DOCKER_HUB_SPRING_REPO}   MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}     MYSQL_DATABASE=${MYSQL_DATABASE} BUILD_TAG=${BUILD_TAG} docker compose -f /home/vagrant/dockerComposeDeployment/Docker-Compose-Deployment.yml up -d '
                sh 'unset BUILD_TAG'

              }
            }

    }
}


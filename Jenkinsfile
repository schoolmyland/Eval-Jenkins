pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "poissonchat13" // replace this with your docker-id
DOCKER_IMAGE_CAST = "eval-jenkins-cast"
DOCKER_IMAGE_MOVIE = "eval-jenkins-movie"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker-compose up -d
                    sleep 10
                    '''
                    }
                }
            }
        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost:8002/api/v1/casts/docs
                    curl localhost:8001/api/v1/movies/docs
                    '''
                    }
            }

        }
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker-compose down
                 sleep 15
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service/
                 sleep 6
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service/
                 sleep 6
                 '''
                }
            }
        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                docker tag $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG $DOCKER_ID/$DOCKER_IMAGE_CAST:latest
                docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:latest
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                docker tag $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG $DOCKER_ID/$DOCKER_IMAGE_MOVIE:latest
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:latest
                '''
                }
            }

        }
stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp ./cast-movie-services/dev-value.yaml ./values.yaml
                cat values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install app cast-movie-services --values=values.yaml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en QA'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp ./cast-service/qa-value.yaml cs-values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app cast-movie-services --values=values.yml --namespace qa
                '''
                }
            }

        }		
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp ./cast-service/staging-value.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app cast-movie-services --values=values.yml --namespace staging
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {

                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Une validation Manuel est necessaire pour le deploiement en production', ok: 'oui'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp ./cast-service/prod-value.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app cast-movie-services --values=values.yml --namespace prod
                '''
                }
            }

        }

}
}

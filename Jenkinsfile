pipeline {
    environment {
        DOCKER_ID = "poissonchat13"
        DOCKER_IMAGE_CAST = "eval-jenkins-cast"
        DOCKER_IMAGE_MOVIE = "eval-jenkins-movie"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any

    stages {
        stage('Docker run') {
            steps {
                script {
                    sh '''
                    docker-compose up -d
                    sleep 10
                    '''
                }
            }
        }
        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                    curl localhost:8002/api/v1/casts/docs
                    curl localhost:8001/api/v1/movies/docs
                    '''
                }
            }
        }
        stage('Docker Build') {
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
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
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
        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
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
        stage('Deploiement en QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp ./cast-movie-services/qa-value.yaml values.yaml
                    cat values.yaml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                    helm upgrade --install app cast-movie-services --values=values.yaml --namespace qa
                    '''
                }
            }
        }		
        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp ./cast-movie-services/prep-value.yaml values.yaml
                    cat values.yaml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                    helm upgrade --install app cast-movie-services --values=values.yaml --namespace prep
                    '''
                }
            }
        }
        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    if (env.GIT_BRANCH == 'master') {
                        timeout(time: 15, unit: "MINUTES") {
                            input message: 'Une validation Manuel est necessaire pour le deploiement en production', ok: 'oui'
                        }
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp ./cast-movie-services/prod-value.yaml values.yaml
                        cat values.yaml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                        helm upgrade --install app cast-movie-services --values=values.yaml --namespace prod
                        '''
                    } else {
                        echo "Branch non master pas de deployment en prod"
                        echo env.BRANCH_NAME
                    }
                }
            }
        }
    }
}

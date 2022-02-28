pipeline {
    environment {
        registry = 'ravitejachowdary/rtcprod'
        registryCredential = 'dockerhub_id'
        devcontext = 'dev-swarm'
        prodcontext = 'prod-swarm'
        devnode = '10.50.1.77'
        prodnode = '10.50.1.245'
        dockerImage = ''
    }
    agent any
    stages {
        stage('Building Prod Docker Branch') {
            when {
                expression {
                    return env.BRANCH_NAME != 'docker-context-prod'
                }
            }
            steps {
                script {
                    dockerImage = docker.build registry + ":v$BUILD_NUMBER"
                }
            }
        }
        stage('Push Image To DockerHUB') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Cleaning up') {
            steps {
                sh "docker rmi $registry:v$BUILD_NUMBER"
            }
        }
        stage('Deploying to Prod Docker Swarm') {
            steps {
                sh "docker context use $prodcontext"
                sh 'docker service rm testing1 || true'
                sh "docker service create --name testing1 -p 8100:80 $registry:v$BUILD_NUMBER"
            }
        }
        stage('Verifying The Deployment') {
            steps {
                sh 'curl http://$prodnode:8100 || exit 1'
            }
        }
        stage('Change Context To Default') {
            steps {
                sh 'docker context use default'
            }
        }
    }
}

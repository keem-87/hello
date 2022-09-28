pipeline {
    agent none 
    environment {
        registry = "rr935981/go_app"
        docker_user = "rr935981"
        docker_app = "go_app"
        GOCACHE = "/tmp"
    }
    stages {
        stage('Build') {
            agent {
                kubernetes {
                    inheritFrom 'golang'
                }
            }
            steps {
                container('golang') {
                    // Create our project directory.
                    sh 'cd ${GOPATH}/src'
                    sh 'mkdir -p ${GOPATH}/src/hello-world'
                    // Copy all files in our Jenkins workspace to our project directory.                
                    sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
                    // Build the app.
                    sh 'export GO111MODULE=auto; go build'  
                }
            }     
        }
        stage('Test') {
            agent {
                kubernetes {
                    inheritFrom 'golang'
                }
            }
            steps {
                container('golang') {                 
                    // Create our project directory.
                    sh 'cd ${GOPATH}/src'
                    sh 'mkdir -p ${GOPATH}/src/hello-world'
                    // Copy all files in our Jenkins workspace to our project directory.                
                    sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
                    // Remove cached test results.
                    sh 'go clean -cache'
                    // Run Unit Tests.
                    sh 'export GO111MODULE=auto; go test ./... -v -short'            
                }
            }
        }
        stage('Publish') {
            agent {
                kubernetes {
                    inheritFrom 'docker'
                }
            }
            steps{
                container('docker') {
                //    sh 'echo $DOCKER_TOKEN | docker login --username $DOCKER_USER --password-stdin'
                //    sh 'docker build -t $DOCKER_REGISTRY:$BUILD_NUMBER .'
                //    sh 'docker push $DOCKER_REGISTRY:$BUILD_NUMBER'
                    sh 'docker build -t 130.127.132.210:31000/go_app:$BUILD_NUMBER .'
                    sh 'docker push 130.127.132.210:31000/go_app:$BUILD_NUMBER'
                }
            }
        }
        stage ('Deploy') {
            agent {
                node {
                    label 'deploy'
                }
            }
            steps {
                sshagent(credentials: ['cloudlab']) {
                    sh "sed -i 's/DOCKER_USER/${docker_user}/g' deployment.yml"
                    sh "sed -i 's/DOCKER_APP/${docker_app}/g' deployment.yml"
                    sh "sed -i 's/BUILD_NUMBER/${BUILD_NUMBER}/g' deployment.yml"
                    sh 'scp -r -v -o StrictHostKeyChecking=no *.yml lngo@155.98.37.91:~/'
                    sh 'ssh -o StrictHostKeyChecking=no lngo@155.98.37.91 kubectl apply -f /users/lngo/deployment.yml -n jenkins'
                    sh 'ssh -o StrictHostKeyChecking=no lngo@155.98.37.91 kubectl apply -f /users/lngo/service.yml -n jenkins'                                        
                }
            }
        }
    }
}

pipeline {
    
    agent any
    
    options {
        // Timeout counter starts AFTER agent is allocated
        timeout(time: 300, unit: 'SECONDS')
    }
    
    environment {
        registry = "eitanas1/web-calculator"
        registryCredential = 'dockerhub_id'
        dockerImage = ''
    }
    
    stages {
        stage('Build App') {
            steps {
                // Build the Go application
                sh """
                    go build -v cmd/main.go
                    mv main docker/web-calculator
                """
            }
        }
        
        stage('Test App') {
            steps {
                // Run Go unit tests
                sh 'go test -v ./...'
            }
        }
        
        stage('Dependency-Check') {
            steps {
                // Invoke OWASP Dependency-Check
                /*dependencyCheck additionalArguments: '--project WORKSPACE', odcInstallation: 'SCA'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'*/
                println "This stage was commented out since it is downloading a lot of data and my AWS free tier program is out of data."
            }
        }
        
        stage('Build Image') {
            steps {
                // Build the Docker image
                dir('./docker') {
                    script {
                        dockerImage = docker.build("${registry}:${BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Image Security Check') {
            steps {
                // Scan image for vulnerabilities
                sh """
                    grype "${registry}:${BUILD_NUMBER}" -v -o table --scope all-layers
                """
            }
        }
        
        stage('Publish Image') {
            steps {
                // Publish the Docker image
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
            sh "docker images"
            sh "docker rmi -f ${registry}:${BUILD_NUMBER} ${registry}:latest"
            // sh 'docker rmi $(docker images -aq)'
            sh "docker images"
        }
    }
}

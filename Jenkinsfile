
pipeline {
    agent any

    environment {
        JAVA_HOME = '/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home'
        PATH = "/Users/piya/.docker/bin:/usr/local/bin:/opt/homebrew/bin:/opt/homebrew/opt/openjdk@17/bin:/opt/homebrew/sbin:$PATH"

    }

    stages {
        stage('Check Tools') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
                sh 'docker --version'
            }
        }

        stage('Build Java App') {
            steps {
                dir('devops-pratice/hello-devops') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('devops-pratice/hello-devops') {
                    sh 'docker build --no-cache -t hello-devops:latest .'
                }
            }
        }

        stage('Test Docker Container') {
            steps {
                dir('devops-pratice/hello-devops') {
                    sh '''
                        OUTPUT=$(docker run --rm hello-devops:latest)
                        echo "Application output: $OUTPUT"
                        test "$OUTPUT" = "Hello World!"
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                            -u "$DOCKERHUB_USER" --password-stdin

                        docker tag hello-devops:latest \
                            "$DOCKERHUB_USER/hello-devops:latest"

                        docker push "$DOCKERHUB_USER/hello-devops:latest"

                        docker logout
                    '''
                }
            }
        }
    }
}


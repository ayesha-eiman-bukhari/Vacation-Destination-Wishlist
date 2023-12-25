pipeline {
    agent any

    stages {
      stage('Build') {
        steps {
          script {
            dockerImage = docker.build("ayeshabukhari/vacation-destinations:${env.BUILD_ID}")
        }
    }
}
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-cred') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html'
            }
        }

        stage('Deploy') {
            steps {
                script {
                   
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "Ayesha's_Terminal", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull ayeshabukhari/vacation-destinations:${env.BUILD_ID}
                                        docker stop vacation-destinations-container || true
                                        docker rm vacation-destinations-container || true
                                        docker run -d --name vacation-destinations-container -p 80:80 ayeshabukhari/vacation-destination:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                  
                    boolean isDeploymentSuccessful = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://13.48.48.182:80', returnStdout: true).trim() == '200'

                    if (!isDeploymentSuccessful) {
                       
                        def previousSuccessfulTag = readFile('previous_successful_tag.txt').trim()
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "Ayesha's_Terminal",
                                    transfers: [sshTransfer(
                                        execCommand: """
                                            docker pull ayeshabukhari/vacation-destination:${previousSuccessfulTag}
                                            docker stop vacation-destination-container || true
                                            docker rm vacation-destination-container || true
                                            docker run -d --name vacation-destination-container -p 80:80 ayeshabukhari/vacation-destination:${previousSuccessfulTag}
                                        """
                                    )]
                                )
                            ]
                        )
                    } else {
                       
                        writeFile file: 'previous_successful_tag.txt', text: "${env.BUILD_ID}"
                    }
                }
            }
        }
    }

    post {
        failure {
            mail(
                to: 'sp20-bcs-021@cuiatk.edu.pk',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: """Something is wrong with the build ${env.BUILD_URL}
                Rolling back to the previous version
                Regards,
                Jenkins
                """
            )
        }
    }
}

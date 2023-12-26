pipeline {
    agent any

    stages {
      stage('Build') {
        steps {
          script {
            dockerImage = docker.build("meeyan/distance-converter:${env.BUILD_ID}")
        }
    }
}
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', '4d46a31c-a4c7-454a-a4f2-62e664ce28d4') {
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
                                configName: "jenkins-demo", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull meeyan/distance-converter:${env.BUILD_ID}
                                        docker stop distance-converter-container || true
                                        docker rm distance-converter-container || true
                                        docker run -d --name distance-converter-container -p 80:80 meeyan/distance-converter:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                  
                    boolean isDeploymentSuccessful = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://13.211.147.129:80', returnStdout: true).trim() == '200'

                    if (!isDeploymentSuccessful) {
                       
                        def previousSuccessfulTag = readFile('previous_successful_tag.txt').trim()
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "jenkins-demo",
                                    transfers: [sshTransfer(
                                        execCommand: """
                                            docker pull meeyan/distance-converter:${previousSuccessfulTag}
                                            docker stop distance-converter-container || true
                                            docker rm distance-converter-container || true
                                            docker run -d --name distance-converter-container -p 80:80 meeyan/distance-converter:${previousSuccessfulTag}
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
                to: 'fa20-bse-038@cuiatk.edu.pk',
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

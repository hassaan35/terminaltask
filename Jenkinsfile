pipeline {
    agent any

    stages {
      stage('Build') {
        steps {
          script {
            dockerImage = docker.build("meeyan/meeyan-cv:${env.BUILD_ID}")
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
                sh 'ls -l index.html' // Simple check for index.html
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy the new version
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "jenkins-demo", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull meeyan/meeyan-cv:${env.BUILD_ID}
                                        docker stop meeyan-cv-container || true
                                        docker rm meeyan-cv-container || true
                                        docker run -d --name meeyan-cv-container -p 80:80 meeyan/meeyan-cv:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                    // Check if deployment is successful
                    boolean isDeploymentSuccessful = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http:// 3.26.23.252:80', returnStdout: true).trim() == '200'

                    if (!isDeploymentSuccessful) {
                        // Rollback to the previous version
                        def previousSuccessfulTag = readFile('previous_successful_tag.txt').trim()
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "jenkins-demo",
                                    transfers: [sshTransfer(
                                        execCommand: """
                                            docker pull meeyan/meeyan-cv:${previousSuccessfulTag}
                                            docker stop meeyan-cv-container || true
                                            docker rm meeyan-cv-container || true
                                            docker run -d --name meeyan-cv-container -p 80:80 Meeyan/meeyan-cv:${previousSuccessfulTag}
                                        """
                                    )]
                                )
                            ]
                        )
                    } else {
                        // Update the last successful tag
                        writeFile file: 'previous_successful_tag.txt', text: "${env.BUILD_ID}"
                    }
                }
            }
        }
    }

    post {
        failure {
           emailext body: 'email sent out from jenkins', subject: 'test email', to: 'hassaan72773@gmail.com'
        }
    }
}

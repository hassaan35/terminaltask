pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("meeyan/personal-portfolio:${env.BUILD_ID}")
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
                                        docker pull meeyan/personal-portfolio:${env.BUILD_ID}
                                        docker stop personal-portfolio-container || true
                                        docker rm personal-portfolio-container || true
                                        docker run -d --name personal-portfolio-container -p 80:80 meeyan/personal-portfolio:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                    
    post {
        failure {
            mail(
                to: 'fa20-bse-038@cuiatk.edu.pk',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
    }
}

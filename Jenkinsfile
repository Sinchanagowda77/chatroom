pipeline{
    agent any
    tools{
        maven 'maven-3.9.10'
    }
    stages{
        stage('clone'){
            steps{
                git 'https://github.com/Sinchanagowda77/chatroom.git'
            }
        }
        stage('validate'){
            steps{
                sh 'mvn --version'
                sh 'mvn clean validate'
            }
        }
        stage('compile'){
            steps{
                sh 'mvn compile'
            }
        }

        stage('trivy'){
            steps{
                sh 'trivy fs --format json --output trivy-fs-result.json .'
            }
        }
        stage('package'){
            steps{
                sh 'mvn package'
            }
        }
        post {
         always {
            script {
                // Determine color based on build status
                def color = currentBuild.currentResult == 'SUCCESS' ? 'green' : 'red'

                // Send the email
                emailext(
                    subject: "Jenkins Build Notification - ${currentBuild.fullDisplayName}",
                    body: """
                        <h2 style="color:${color};">Build Notification</h2>
                        <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                        <p><strong>Status:</strong> <span style="color:${color};">${currentBuild.currentResult}</span></p>
                        <p><strong>View build:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                    """,
                    to: 'charanv369@gmail.com',
                    from: 'jenkins@gmail.com',
                    replyTo: 'jenkins@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-fs-result.html'
                )
            }
        }
        
        }
    
    }

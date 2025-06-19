pipeline{
    agent any
    tools{
        maven 'maven'
    } 
    stages{
        stage('clone'){
            steps{
                git 'https://github.com/VootlaSaiCharan/chatroom.git'
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
        stage('trivy-file-scan'){
            steps{
                sh 'trivy fs --severity HIGH,CRITICAL --format json --output trivy-fs-result.json .'
                sh ''' trivy convert \
                --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                -o trivy-fs-result.html trivy-fs-result.json '''
            }
        }
        stage('package'){
            steps{
                sh 'mvn package'
            }
        }
        stage('deploy'){
            steps{
                sh 'cp -r target/chatroom-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps/ROOT.war'
            }
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
                    to: 'sinchanagowda204@gmail.com',
                    from: 'jenkins@gmail.com',
                    replyTo: 'jenkins@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-fs-result.html'
                )
            }
        }
    }
}
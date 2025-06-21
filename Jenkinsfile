
pipeline{
    agent any
    tools{
        maven 'maven'
    } 
    environment {
        SONARQUBE_HOME = tool 'sonarqube-scanner' 
    }
    stages{
        stage('clone'){
            steps{
                git 'https://github.com/lavanyavasudev/chatroom.git'
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
                sh 'trivy fs --severity UNKNOWN --exit-code 1 --format json --output trivy-fs-result.json .'
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
        stage('sonarqube analysis'){
            steps{
                // Print the SONARQUBE_HOME variable for debugging
                sh 'echo "$SONARQUBE_HOME"'
                // Run SonarQube scanner with the specified project key and name
                withSonarQubeEnv('sonarqube-server'){
                    sh ''' $SONARQUBE_HOME/bin/sonar-scanner -Dsonar.projectKey=chatroom \
                        -Dsonar.projectName=chatroom -Dsonar.java.binaries=target '''
                }
            }
        }
        stage('sonarqube quality gate'){
            steps{
                timeout(time: 60, unit: 'SECONDS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-cred'
                }
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

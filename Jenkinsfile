pipeline {
    agent any 
    environment{
        SONAR_HOME = tool "Sonar"
    }
    stages {
        stage ("Code clone") {
           steps{
              git url: "https://github.com/sheshdhar3/Node-todo-cicd-App" , branch: "master"
              echo "Code Cloned sucessfully"
            }
        }
        stage ("SonarQube Analysis"){
            steps{
                withSonarQubeEnv("Sonar"){
                sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=PQRNodeApp-CICD -Dsonar.projectKey=PQRNodeApp-CICD"
                }
            }
        }
        stage ("SonarQube Quality Gate checks"){
            steps{
                timeout(time: 1, unit: "MINUTES") {
                waitForQualityGate abortPipeline: false
                }
            }
        }
        stage ("OWASP"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                echo "OWASP check completed"
            }
        }
        stage ("Build and Test Stage"){
            steps{
                sh 'docker build -t node-todo-cicd:final .'
                echo "Code build successfully"
            }    
        }
        stage ("Trivy scan"){
            steps{
                sh "trivy image node-todo-cicd:final"
                
            }
        }
        stage ("Push to private docker hub repository"){
            steps{
                withCredentials([usernamePassword(credentialsId:"Dockerhubcreds",passwordVariable:"dockerPassword",usernameVariable:"dockerUser")]){
                sh "docker login -u ${env.dockerUser} -p ${env.dockerPassword}"
                sh "docker tag node-todo-cicd:final ${env.dockerUser}/node-todo-cicd:final"
                sh "docker push ${env.dockerUser}/node-todo-cicd:final"
                echo "Image pushed on dockerhub repository"
                }
            }
        }
        stage ("Deploy stage"){
            steps{
                sh "docker-compose down && docker-compose up -d"
                echo "Apps Deployed Successfully"
            }
        }
    }
    post {
        always {
            script {
                emailext(
                    subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
                    body: """${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${currentBuild.currentResult} \n
                    Check console output at ${env.BUILD_URL} to view the results.""",
                    recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'DevelopersRecipientProvider']],
                    to: 'sheshdevops@gmail.com'
                )
            }
        }
    }
}

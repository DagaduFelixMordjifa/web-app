 COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline{
    agent any
    tools {
        maven "maven3.9.6"
    }

    stages{
        stage ("Git clone") {
            steps{
                git branch: 'main', url: 'https://github.com/DagaduFelixMordjifa/web-app.git'
            }
        }

          stage("Build with Maven"){
            steps{
                sh "mvn clean"
            }
        }

         stage("Testing with Maven"){
            steps {
                sh "mvn test"
            }
        }

        stage("Package with Maven"){
            steps {
                sh "mvn package"
            }
        }  

         stage('SonarQube Analysis') {
            environment {
                ScannerHome = tool 'sonar5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage("Upload to Nexus"){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/First-pipeline-job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '52.15.63.123:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-snapshot', version: '3.1.2-SNAPSHOT'
            }
        }

        stage("Deploy to UAT"){
            steps{
                deploy adapters: [tomcat9(credentialsId: 'tomcat-id', path: '', url: 'http://3.138.123.208:8080')], contextPath: null, war: 'target/*.war'
            }
        }
    }

    post {
        always {
            slackSend channel: 'team3-africa', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}"
        }
    }
    
}
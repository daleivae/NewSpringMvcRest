pipeline {
    environment {
        SLACK_CHANNEL = "#notificaciones-desde-jenkins"
        SLACK_TEAM_DOMAIN = "sustantiva-jenkins"
        SLACK_TOKEN = credentials("token-slack")
    }
    agent any
    stages {
        stage('Initialize'){
            steps{
                echo "Esta es el inicio"
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B package'
            
            }
        }
            
        stage('Test') {
            steps {
                 sh "mvn clean verify sonar:sonar -Dsonar.projectKey=ScannerMavenJenkins \
                                                  -Dsonar.host.url=http://172.28.224.1:8082 \
                                                  -Dsonar.login=sqp_98788e87facf9eeb7295ce9a2ebc2c54bf292101" 
            
            }
        } 
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: "nexus3",
                            protocol: "http",
                            nexusUrl: "172.28.224.1:8081/repository/maven-releases/",
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: "MiForkProyectoMaven",
                            credentialsId: "Nexus",
                            artifacts: [
                                [artifactId: pom.artifactId,
                                        classifier: '',
                                        file: artifactPath,
                                        type: pom.packaging]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        
            } 
     }
    post {
        always {
                slackSend message:"Regardless of the completion status of the Pipeline or stage run - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        changed {
                slackSend message:"the current Pipeline run has a different completion status from its previous run - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        fixed {
                slackSend message:"The current Pipeline run is successful and the previous run failed or was unstable - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        regression {
                slackSend message:"The current Pipeline or status is failure, unstable, or aborted and the previous run was successful - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        aborted {
                slackSend message:"The current Pipeline run has an <aborted> status was successful - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        failure {
                slackSend failOnError:true message:"The current Pipeline or stage run has a <failed> status  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
        success {
                slackSend message:"The current Pipeline or stage run has a <success> status - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        unstable {
                slackSend message:"The current Pipeline run has an <unstable> status - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        unsuccessful {
                slackSend message:"The current Pipeline or stage run has not a <success> status - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        cleanup {
                slackSend message:"Every other post condition has been evaluated - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
    }
}

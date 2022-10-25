pipeline {
    agent any
    tools {
        maven "maven"
    }
    stages{
        stage ("Unit test with mvn"){
            steps{
                script{
                    sh "mvn test"
                }
            }
        }
        stage ("Build with mvn"){
            steps{
                script{
                    sh "mvn package"
                }
            }
        }
        stage ("Test with sonar"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqubeconfigured') {
                     sh "mvn sonar:sonar"
                   }
                }
            }
        }
        stage ("Sonarqube quality gate"){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqubeconfigured'
                }
            }
        }
        stage("Upload jar to nexus"){
            steps{
                script{
                    def readPomVersion = readMavenPom file: 'pom.xml'

                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "springboot-snapshot" : "springboot-release"
                    nexusArtifactUploader artifacts: 
                    [
                        [
                            artifactId: 'spring-boot-mongo', 
                            classifier: '', file: 'target/spring-boot-mongo-1.0.jar', 
                            type: 'jar']
                            ], 
                            credentialsId: 'e161dcd8-ce01-45f9-b023-efc0ea5b6a58', 
                            groupId: 'com.example', 
                            nexusUrl: '10.0.0.168:4000', 
                            nexusVersion: 'nexus3', 
                            protocol: 'http', 
                            repository: nexusRepo, 
                            version: "${readPomVersion.version}"
                }
            }
        }
        stage("docker build"){
            steps{
                script{
                    sh "docker build . -t $JOB_NAME:v1.$BUILD_ID"
                    sh "docker tag $JOB_NAME:v1.$BUILD_ID tolux17tech/$JOB_NAME:v1.$BUILD_ID"
                }
            }
        }
        stage("docker push") {
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS |docker login -u $USER --password-stdin"
                    sh "docker push tolux17tech/$JOB_NAME:v1.$BUILD_ID"
                   }
                }
            }
        }
        stage("docker run"){
            steps{
                script{
                    echo "coming soon"
                   
                    sh "docker run -d -p2000:8080 tolux17tech/$JOB_NAME:v1.$BUILD_ID"
                }
            }
        }
    }
}

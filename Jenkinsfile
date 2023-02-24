@Library("Shared-Library") _
pipeline{
    agent any
    environment {
    DOCKER_HUB_CREDENTIALS = credentials('dockerhubpwd')
    DOCKER_FILE_PATH = './Dockerfile'
    DOCKER_IMAGE_NAME = 'kranthikg/sharedlibimage'
    DOCKER_IMAGE_TAG = "${DOCKER_IMAGE_NAME}:${BUILD_ID}"
  }
    
    stages{
      stage('Sample_Stage'){
          steps{
              script{
                  welcome.sample('Jenkins Shared-Libraries')
              }
          }
      }
      stage('S_GIT_Checkout') {
        steps {
          script{
                    fetchCode.GitFetch()
                }
            }   
        }
      stage('Build Code'){
          steps{
              script{
                  maven.mavenClean()
                  maven.mavenTest()
                  maven.mavenCleanInstall()
                }
            }
        }
      stage('Upload to Nexus') {
            steps {
                script {
                       nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: '/var/lib/jenkins/.m2/repository/com/javatechie/devops-integration/0.0.1-SNAPSHOT/devops-integration-0.0.1-SNAPSHOT.jar', type: 'jar']], credentialsId: 'NEXUSCRED', groupId: 'com.kranthi', nexusUrl: '170.187.233.234:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'repository', version: '0.0.1-SNAPSHOT'
                    // def artifact = new File('/var/lib/jenkins/.m2/repository/com/javatechie/devops-integration/0.0.1-SNAPSHOT/devops-integration-0.0.1-SNAPSHOT.jar')
                    // nexusUtils.uploadMavenArtifactToNexus('com.javatechie', 'devops-integration', '0.0.1-SNAPSHOT', artifact, 'http://170.187.233.234:8081', 'NEXUSCRED', 'repository')
                }
            }
        }
       stage('docker login'){
           steps{
               script{
                   withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u kranthikg -p ${dockerhubpwd}'
}
               }
           }
       }
       stage('docker build and push'){
           steps{
               script{
                   dockerBuild.build(DOCKER_IMAGE_NAME)
                   dockerBuild.tag(DOCKER_IMAGE_NAME,DOCKER_IMAGE_TAG)
                   dockerBuild.push(DOCKER_IMAGE_TAG)
               }
           }
       }
       stage('update deployment'){
           steps{
               script{
                        sh "cat Deployment.yml"
                        sh "git config user.email kranthikirang@gmail.com"
                        sh "git config user.name KranthiKG"
                        sh "sed -i 's+kranthikg/sharedlibimage.*+${DOCKER_IMAGE_TAG}+g' Deployment.yml"
                        sh "cat Deployment.yml"
                        sh "git add ."
                        sh "git commit -m 'Done by Jenkins Job changemanifest: ${env.BUILD_NUMBER}'"
                        withCredentials([gitUsernamePassword(credentialsId: 'githubcred', gitToolName: 'Default')]) {
                        sh "git push https://github.com/KranthiKG/Devops-Integration-SharedLib.git HEAD:main"
}
                       
               }
           }
       }
    }
}

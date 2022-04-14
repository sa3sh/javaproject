pipeline{
    agent any
    tools {
        maven 'maven'
        jdk "java11"
    }   
    environment {
        def pom = readMavenPom file: 'pom.xml'

        pom_version_array=pom.groupId.split('com.')
        groupID="${pom_version_array[1]}" // devops-mentor
        SONAR_URL ="http://18.232.174.123:9000/"   //credentials('SONAR_URL') //"http://54.209.51.175:9000"
        SONAR_LOGIN_KEY=credentials('sonartoken')
        SONAR_PROJECT="sonarproject"
        ARTIFACTID="${pom.artifactId}"
        GROUPID="${pom.groupId}" //com.devops-mentor
        VERSION="${pom.version}"
        NEXUS_USER=credentials('NEXUS_USER') 
        NEXUS_PASSWORD= credentials('NEXUS_PASSWORD') 
        NEXUS_PROJECT_NAME="javanexusrepo"
        NEXUS_ARTIFACT_URL="http://18.212.90.165:8081//repository/${NEXUS_PROJECT_NAME}/${groupID}/${ARTIFACTID}/${VERSION}/${ARTIFACTID}-${VERSION}.war"
        NEXUS_ARTIFACT_FILE_PATH="app/target/app.war"
        TOMCAT_URL="http://3.95.18.20:8080/"
    }
    

    
    stages {

        stage("pull SCM"){
            steps{
            git branch: 'main', url: 'https://github.com/sa3sh/javaproject.git'   
               }
        }
       
       
          stage("Test"){
           
            steps{
                sh "mvn clean test" 
            }
        }

          stage("Build"){
           
            steps{
                sh "mvn clean install" 
            }
        }
 stage("Code Analysis by Sonar"){
            
            steps{
      
           sh "mvn sonar:sonar \
  -Dsonar.projectKey=${SONAR_PROJECT} \
  -Dsonar.host.url=${SONAR_URL} \
  -Dsonar.login=${SONAR_LOGIN_KEY}"
              }
        }
        stage("Upload to Nexus"){
           
            steps{
nexusPublisher nexusInstanceId: env.NEXUS_PROJECT_NAME, nexusRepositoryId: env.NEXUS_PROJECT_NAME, packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath:env.NEXUS_ARTIFACT_FILE_PATH]], mavenCoordinate: [artifactId: env.ARTIFACTID, groupId:env.GROUPID, packaging: 'war', version: env.VERSION]]]        }

        }
        stage("Pull Artifact"){

            steps{
                sh "wget --user=${NEXUS_USER} --password=${NEXUS_PASSWORD} ${NEXUS_ARTIFACT_URL}"
                
            }
        }
        
          stage("deploy to Server"){
           
            steps{
deploy adapters: [tomcat9(credentialsId: 'tomactdeployer_logindetails', path: '', url: env.TOMCAT_URL )], contextPath: null, war: '**/*.war'

}
        }



}
}

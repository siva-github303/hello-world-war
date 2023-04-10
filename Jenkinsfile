pipeline {
    agent any
    tools {
        maven "LocalMaven"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "demo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
    }
   parameters { booleanParam(name: 'skip_stage', defaultValue: true, description: 'Set to true to skip the stage') }
    stages {
        stage("Maven Build") {
            when { expression { params.skip_stage != true } }
            steps {
                script {
                    bat "mvn package -DskipTests=true"
                }
            }
        }
    stage('SonarCloud') {
  environment {
    SCANNER_HOME = tool 'SonarQubeScanner'
    ORGANIZATION = "Suresh051"
    PROJECT_NAME = "Suresh051_hello-world-war"
  }
  steps {
    withSonarQubeEnv('SonarCloudOne') {
        bat '''
        $SCANNER_HOME/bin/sonar-scanner -Dsonar.organization=$ORGANIZATION \
        -Dsonar.projectKey=$PROJECT_NAME \
        -Dsonar.sources=.
        '''
    }
  }
}
        stage("Publish to Nexus Repository Manager") {
             when { expression { params.skip_stage != true } }
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "**/target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    }
                    else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
            
        post { 
        always { 
            cleanWs()
        }
    }
        }
        
        stage ('Nexus repository download') {
            when { expression { params.skip_stage != true } }
            steps {
            bat '''
            curl -L -o hello-world-war-1.0.0.war -s -X GET "http://localhost:8081/service/rest/v1/search/assets/download?sort=version&repository=demo&maven.groupId=com.efsavage&maven.artifactId=hello-world-war&maven.baseVersion=1.0-SNAPSHOT&maven.extension=war" -H "accept: application/json"
            '''
                
            }
        }
        stage ('deploy to tomcat') {
            when { expression { params.skip_stage != true } }
            steps {            
               bat '''
               cd "C:/Program Files/Apache Software Foundation/Tomcat 9.0/webapps/"
               del "hello-world-war-1.0.0.war"  
               echo y | rmdir /s hello-world-war-1.0.0
               
               cd "C:/Program Files/Apache Software Foundation/Tomcat 9.0/bin"
               Tomcat9.exe stop
               echo %WORKSPACE%
               cd "%WORKSPACE%
               xcopy hello-world-war-1.0.0.war "C:/Program Files/Apache Software Foundation/Tomcat 9.0/webapps"
               cd "C:/Program Files/Apache Software Foundation/Tomcat 9.0/bin"
               Tomcat9.exe start
               '''
        }
    }
    }
}

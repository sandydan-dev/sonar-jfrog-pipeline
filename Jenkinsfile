// Define the URL of the Artifactory registry
def registry = 'https://trial28jeay.jfrog.io/'

pipeline {

  agent any

  environment {
     PATH="/opt/maven/bin:$PATH"
  }

  stages {

     // build start
     stage("build"){

        echo "----- build starts -----"
        sh "mvn clean package -Dmaven.test.skip=true"  // create end project, and skip testing
        echo "----- build end -----" 

     }
     // build end

     // test start
     stage("test"){
        echo "----- test start -----"
        sh "mvn surefire-report:report"  // test java code, testing code data
        echo "----- test end ------"
     }
     // test end


     // start soanrqube analysis
     stage("Sonar Analysis"){

        // sonar tool scanner name
        environment {
          scannerHome= tool "sandy-sonar-scanner"
        }       

        // sonar server
        steps {
          withSonarQubeEnv("sandy-sonar-server"){
            echo "----- start server -----"
            sh "${scannerHome}/bin/sonar-scanner"
            echo "----- end server -----"
          }
        }

     }

     // JFrog start
        
     stage("Jar Publish"){
     
       steps {
         echo "----- jar publish start -----"

         def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "sonarid"
         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
        // rename for version dynamically 
         def version = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()
         
          def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "target/*.war",
                              "target": "sandyjfrog-libs-release-local/war-files-verions/${version}/",
                              "flat": "true",
                              "props": "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""

           def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)

         echo "----- jar publish end -----"
       }
  
     }  

     // Jfrog end


  }
  

}

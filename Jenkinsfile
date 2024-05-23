def registry = 'https://mask9147.jfrog.io'
def imageName = 'mask9147.jfrog.io/mask9147-docker-local/sample_app'
def version   = '2.1.2'
pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
    // Artifactory server details
    ARTIFACTORY_SERVER = 'artifactory-server_id' // Replace with your Artifactory server ID in Jenkins
    ARTIFACTORY_URL = 'https://mask9147.jfrog.io/artifactory' // Replace with your Artifactory URL
    ARTIFACTORY_REPO = 'mavenrepo-libs-release' // Replace with your target repository
    ARTIFACTORY_CREDENTIALS = 'JFROG_ARTIFACORY_UI' // Replace with your Jenkins credential ID    
}
    stages {
        stage("build"){
            steps {
                 echo "----------- build started ----------"
                 sh 'mvn clean deploy -Dmaven.test.skip=true'
                 echo "----------- build complted ----------"
            }
        }
        stage("test"){
            steps{
                echo "----------- unit test started ----------"
                sh 'mvn surefire-report:report'
                 echo "----------- unit test Complted ----------"
            }
        }
        stage('SonarQube analysis') {
        environment {
          scannerHome = tool 'SonarQubeScanner5'
        }
            steps{
            withSonarQubeEnv('SonarQubeScanner3') {
              sh "${scannerHome}/bin/sonar-scanner"
            }
            }
        }
        stage("Quality Gate"){
            steps {
                script {
                timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
            error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        }
        }
            }
        }
        stage('Jar Publish') {
            steps {
                echo "----------- publish started ----------"
                script {
                    // Define the Artifactory server connection
                    def server = Artifactory.server(ARTIFACTORY_SERVER)

                    // Define the Maven resolver and deployer
                    def rtMaven = Artifactory.newMavenBuild()
                    rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
                    rtMaven.deployer server: server, releaseRepo: ARTIFACTORY_REPO, snapshotRepo: ARTIFACTORY_REPO

                    // Run the Maven build
                    rtMaven.run pom: 'pom.xml', goals: 'clean install'

                    // Upload the build-info to Artifactory
                    server.publishBuildInfo(rtMaven.getBuildInfo())
                }
               echo "----------- publish ended ----------"
            }
        }
    stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

            stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'jfrog_cred'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }

    // stage (" Deploy "){
    //     steps {
    //         script {
    //            sh './deploy.sh'  
    //         }
    //     }
    // }

stage(" Deploy ") {
       steps {
         script {
            echo '<--------------- Helm Deploy Started --------------->'
            sh 'helm install sample-app sample-app-1.0.1'
            echo '<--------------- Helm deploy Ends --------------->'
         }
       }
}  
}
}



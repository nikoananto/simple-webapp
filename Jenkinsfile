pipeline {
    agent {
      node {
        label 'webapp-agent' 
      }
    }

    environment {
        version = 'v1'
    }

    stages {
        stage('Preparation') {
            steps {
              container('webapp-agent') {
                echo "App Version = ${version}"
              }
            }
        }

        stage('Test') {
            steps {
              container('webapp-agent') {
                sh "html5validator html/index.html"
              }
            }

            post {
                success {
                   echo "Test Successful"
                }
                failure {
                   echo "Test Failed"
                }
            }
        }

        stage('Build') {
           steps {
              container('openshift-client') {
              sh "git clone https://github.com/atwatanmalikm/simple-webapp.git"
              sh "sed -i \"s|webapp:|webapp:${version}|g\" simple-webapp/manifest/buildconfig.yaml"
              sh "sed -i \"s|webapp:|webapp:${version}|g\" simple-webapp/manifest/app.yaml"
              sh "cat simple-webapp/manifest/buildconfig.yaml simple-webapp/manifest/app.yaml"
              script {
                openshift.withCluster() { 
                  openshift.withProject("jenkins") {
                    openshift.apply("-f simple-webapp/manifest/buildconfig.yaml")
                    openshift.startBuild("simple-webapp","--wait")
                  }
                }
              }
              }
            }
            post {
                success {
                    echo "Build Successful"
                }
                failure {
                   container('openshift-client') {
                   script {
                     openshift.withCluster() {
                       openshift.withProject("jenkins") {
                          openshift.delete("-f simple-webapp/manifest/buildconfig.yaml")
                       }
                     }
                   }
                   }
                }
            }
        }

        stage('Approval') {
          steps {
             input(
               message: "Deploy application with ${version} version to production ?",
               ok: 'Yes, deploy it'
             )
          }
        }

        stage('Deploy') {
           steps {
              container('openshift-client'){
              script {
                openshift.withCluster() {
                  openshift.withProject("jenkins") {
                    openshift.apply("-f simple-webapp/manifest/app.yaml")
                  }
                }
              }
              }
           }
           post {
                success {
                   echo "Deploy Successful"
                }
                failure {
                   echo "Deploy Failed" 
                }
            }
        }
    }
}

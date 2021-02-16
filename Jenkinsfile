pipeline {
  agent {
    docker {
      image 'node:6-alpine'
      args '-p 3000:3000'
    }

  }
  stages {
    stage('Build') {
      steps {
        sh 'npm install'
      }
    }

    stage('Test') {
      parallel {
        stage('Test') {
          environment {
            CI = 'true'
          }
          steps {
            sh './jenkins/scripts/test.sh'
          }
        }

        stage('Unit test') {
          steps {
            echo 'Hello from unit test'
            script {
              jenkins.model.Jenkins.getInstance().getUpdateCenter().getSites().each {
                site -> site.updateDirectlyNow(hudson.model.DownloadService.signatureCheck)
              }

              hudson.model.DownloadService.Downloadable.all().each {
                downloadable -> downloadable.updateNow();
              }

              def plugins = jenkins.model.Jenkins.instance.pluginManager.activePlugins.findAll {
                it.hasUpdate()
              }.collect {
                it.getShortName()
              }

              println "Plugins to upgrade: ${plugins}"
            }

          }
        }

        stage('Performance testing') {
          steps {
            sleep 10
          }
        }

      }
    }

    stage('Deliver') {
      steps {
        sh './jenkins/scripts/deliver.sh'
        input 'Finished using the web site? (Click "Proceed" to continue)'
        sh './jenkins/scripts/kill.sh'
      }
    }

  }
}
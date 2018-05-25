pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr:'10'))
  }
  stages {
      stage('SCP app-common') {
          steps {
              sh 'scp app-common cisd@its93.pvt.hawaii.edu:/usr/local/bin/appDaemon/app-common'
              sh 'scp app-common cisd@its94.pvt.hawaii.edu:/usr/local/bin/appDaemon/app-common'
              sh 'scp app-common cisd@its95.pvt.hawaii.edu:/usr/local/bin/appDaemon/app-common'
              sh 'scp app-common cisd@its96.pvt.hawaii.edu:/usr/local/bin/appDaemon/app-common'
              sh 'scp app-common cisd@its97.pvt.hawaii.edu:/usr/local/bin/appDaemon/app-common'
              sh 'scp app-common cisd@its98.pvt.hawaii.edu:/usr/local/bin/appDaemon/app-common'
              sh 'scp app-common cisd@its99.pvt.hawaii.edu:/usr/local/bin/appDaemon/app-common'
          }
      }
  }
  post {
    failure {
        mail to: 'cahana@hawaii.edu',
             subject: "Failed ${currentBuild.fullDisplayName}",
             body: "${env.RUN_DISPLAY_URL}"
    }
  }
}

pipeline {
  agent {
    node {
      label 'apache_builds'
    }
  }
  stages {
    stage('prepareRepos') {
      agent {
        node {
          label 'apache_builds'
        }
      }
      steps {
        sh '${WORKSPACE}/scripts/prepare_repos.sh'
      }
    }
    stage('checkLicenses') {
      agent {
        node {
          label 'apache_builds'
        }
      }
      steps {
        sh '${WORKSPACE}/scripts/check_licenses.sh'
      }
    }
    stage('setupVBoxSVC') {
      agent {
        node {
          label 'apache_builds'
        }
      }
      steps {
        script {
          withEnv(['JENKINS_NODE_COOKIE=dontkill']) {
            sh '${WORKSPACE}/scripts/setup_svc.sh'
          }
        }
      }
    }
    stage('build') {
      parallel {
        stage('BuildUbuntu') {
          agent {
            node {
              label 'apache_builds'
            }
          }
          steps {
            sh '${WORKSPACE}/scripts/run_test.sh ubuntu'
            sh 'cp -r out out-$BUILD_NUMBER'
          }
          post {
            always {
              stash(name: "ubuntu-${env.BUILD_NUMBER}", includes: "out-${env.BUILD_NUMBER}/*.xml")
              sh 'rm -r out-$BUILD_NUMBER'
              sh 'rm out/*.xml'
              sh '${WORKSPACE}/scripts/shutdown.sh ubuntu-$BUILD_NUMBER'
            }
          }
        }
        stage('BuildCentos') {
          agent {
            node {
              label 'apache_builds'
            }
          }
          steps {
            sh '${WORKSPACE}/scripts/run_test.sh centos'
            sh 'cp -r out out-$BUILD_NUMBER'
          }
          post {
            always {
              stash(name: "centos-${env.BUILD_NUMBER}", includes: "out-${env.BUILD_NUMBER}/*.xml")
              sh 'rm -r out-$BUILD_NUMBER'
              sh 'rm out/centos.xml'
              sh '${WORKSPACE}/scripts/shutdown.sh centos-$BUILD_NUMBER'
            }
          }
        }
      }
    }
    stage('cleanup') {
      agent {
        node {
          label 'apache_builds'
        }
      }
      steps {
        sh '${WORKSPACE}/scripts/clean_branches.sh'
      }
    }
  }
  post {
    always {
      unstash "ubuntu-${env.BUILD_NUMBER}"
      unstash "centos-${env.BUILD_NUMBER}"
      junit "out-${env.BUILD_NUMBER}/*.xml,out-${env.BUILD_NUMBER}/centos.xml"
      sh 'rm -r out-$BUILD_NUMBER'
    }
  }
}

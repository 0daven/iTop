pipeline {
  agent any
    parameters {
        booleanParam(name: 'debugMode', defaultValue: 'false', description: 'Debug mode?')
        booleanParam(name: 'runNonRegOQLTests', defaultValue: 'false', description: 'Do You want to run legacy OQL regression tests?')
    }
  stages {

    stage('init') {
      parallel {
        stage('debug') {
          steps {
            sh './.jenkins/bin/init/debug.sh'
          }
        }
        stage('append files to project') {
          steps {
            sh './.jenkins/bin/init/append_files.sh'
          }
        }
        stage('composer install') {
          steps {
            sh './.jenkins/bin/init/composer_install.sh'
          }
        }
      }
    }

    stage('unattended_install') {
      parallel {
        stage('unattended_install default env') {
          steps {
            sh './.jenkins/bin/unattended_install/default_env.sh'
          }
        }
      }
    }

    stage('test') {
      parallel {
        stage('composer') {
          steps {
            sh './.jenkins/bin/tests//dependencies_outdated.sh'

            sh './.jenkins/bin/tests//security_check.sh'
          }
        }
        stage('phpunit') {
          steps {
            sh './.jenkins/bin/tests/phpunit.sh ${debugMode} ${runNonRegOQLTests}'
          }
        }
      }
    }
  }

  post {
    always {
      junit 'var/test/phpunit-log.junit.xml'
                archiveArtifacts(artifacts: 'var/test/listOutdated.log', allowEmptyArchive: true)
    }
    failure {
      slackSend(channel: "#jenkins-itop", color: '#FF0000', message: "Ho no! Build failed! (${currentBuild.result}), Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      archiveArtifacts(artifacts: 'var/test/security_check.log', allowEmptyArchive: true)
    }
    fixed {
      slackSend(channel: "#jenkins-itop", color: '#FFa500', message: "Yes! Build repaired! (${currentBuild.result}), Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
  }

  environment {
    DEBUG_UNIT_TEST = '0'
  }
  options {
    timeout(time: 20, unit: 'MINUTES')
  }
}

pipeline {
    agent {
      label "jenkins-maven"
    }
    environment {
      ORG               = 'activiti'
      APP_NAME          = 'activiti-cloud-example-chart'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
      GITHUB_CHARTS_REPO    = "https://github.com/Activiti/activiti-cloud-helm-charts.git"
      GITHUB_HELM_REPO_URL = "https://activiti.github.io/activiti-cloud-helm-charts/"
      HELM_RELEASE_NAME = "example-${BRANCH_NAME}-${BUILD_NUMBER}".toLowerCase()

    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          branch 'PR-*'
        }
        environment {
          PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
          PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
          HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
        }

        steps {
          container('maven') {
           dir ('./charts/activiti-cloud-full-example') {
	           // sh 'make build'
              sh 'make install'
            }
          
            git 'https://github.com/Activiti/activiti-cloud-acceptance-scenarios.git'
            sh 'sleep 120'
            dir ("activiti-cloud-acceptance-scenarios") {
              sh "mvn clean install -DskipTests && mvn -pl '!apps-acceptance-tests,!multiple-runtime-acceptance-tests,!security-policies-acceptance-tests' clean verify"
            }
          }
        }
      }
      stage('Build Release') {
        when {
          branch 'master'
        }
        steps {
          container('maven') {
            // ensure we're not on a detached head
            sh "git checkout master"
            sh "git config --global credential.helper store"

            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"

            dir ('./charts/activiti-cloud-full-example') {
                sh 'make tag'
                sh 'make release'
                sh 'make github'
            }
          }
        }
      }

      stage('Promote to Environments') {
        when {
          branch 'master'
        }
        steps {
          container('maven') {
            dir ('./charts/activiti-cloud-full-example') {
              sh 'jx step changelog --version v\$(cat ../../VERSION)'
              // promote through all 'Auto' promotion Environments
              sh 'jx promote -b --all-auto --helm-repo-url=$GITHUB_HELM_REPO_URL --timeout 1h --version \$(cat ../../VERSION) --no-wait'
             // sh 'cd ../.. && updatebot push-version --kind helm activiti-cloud-full-example \$(cat VERSION)'
            }
          }
        }
      }
    }
   post {
        always {
          container('maven') {
            dir('charts/activiti-cloud-full-example') {
               sh "make delete" 
            }
            sh "kubectl delete namespace $PREVIEW_NAMESPACE" 
          }
          cleanWs()
        }
  }
}

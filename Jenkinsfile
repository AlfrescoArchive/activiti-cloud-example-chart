pipeline {
    options {
      disableConcurrentBuilds()
    }  
    agent {
      label "jenkins-maven-java11"
    }
    environment {
      ORG               = 'activiti'
      APP_NAME          = 'activiti-cloud-full-example'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
      GITHUB_CHARTS_REPO    = "https://github.com/Activiti/activiti-cloud-helm-charts.git"
      GITHUB_HELM_REPO_URL = "https://activiti.github.io/activiti-cloud-helm-charts/"
      HELM_RELEASE_NAME = "example-$BRANCH_NAME-$BUILD_NUMBER".toLowerCase()

      PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
      PREVIEW_NAMESPACE = "example-$BRANCH_NAME-$BUILD_NUMBER".toLowerCase()
      
      REALM = "activiti"

    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          // branch 'PR-*'
          branch 'zz-*'

        }
        environment {
         GATEWAY_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE.35.228.195.195.nip.io"
         SSO_HOST = "activiti-keycloak.$PREVIEW_NAMESPACE.35.228.195.195.nip.io"
        }
        steps {
          container('maven') {
           dir ("./charts/$APP_NAME") {
	           // sh 'make build'
              sh 'make install'
              sh 'pwd'
            }

            // dir("./activiti-cloud-acceptance-scenarios") {
              // sh 'pwd'
              // git 'https://github.com/Activiti/activiti-cloud-acceptance-scenarios.git'
              // sh 'sleep 120'
              // sh "mvn clean install -DskipTests && mvn -pl 'runtime-acceptance-tests' clean verify"
              // sh 'pwd'
            // }
          }
        }
      }
      stage('Build Release') {
        when {
          // branch 'master'
          branch 'PR-*'
        }
	      environment {
         GATEWAY_HOST = "activiti-cloud-gateway.jx-staging.35.228.195.195.nip.io"
         SSO_HOST = "activiti-keycloak.jx-staging.35.228.195.195.nip.io"
        }      
        steps {
          container('maven') {
            // ensure we're not on a detached head
            sh 'pwd'
            sh 'ls'
            // sh "git checkout master"
            sh "git config --global credential.helper store"
            sh "jx step git credentials"
            // so we can retrieve the version in later steps
             sh "echo \$(jx-release-version) > VERSION"
            dir ("./charts/$APP_NAME") {
	           // sh 'make build'
              sh 'make install'
            }
	   //run tests	
            // dir("./activiti-cloud-acceptance-scenarios") {
            //   git 'https://github.com/Activiti/activiti-cloud-acceptance-scenarios.git'
            //   sh 'sleep 120'
            //   sh "mvn clean install -DskipTests && mvn -pl 'runtime-acceptance-tests' clean verify"
            // }	  
	    //end run tests	  
            dir ("./charts/$APP_NAME") {
                sh 'make tag'
                sh 'make release'
                sh 'make github'
            }
          }
        }
      }

      stage('Promote to Environments') {
        when {
          // branch 'master'
          branch 'PR-*'
        }
        steps {
          container('maven') {
            dir ("./charts/$APP_NAME") {
              //sh 'jx step changelog --version v\$(cat ../../VERSION)'
              // promote through all 'Auto' promotion Environments
//commented due to jx .ignore bug
		          //sh 'jx promote -b --all-auto --helm-repo-url=$GITHUB_HELM_REPO_URL --timeout 1h --version \$(cat ../../VERSION) --no-wait'
              sh 'make promote'
            }
          }
        }
      }
    }
   post {
        always {
          container('maven') {
            dir("./charts/$APP_NAME") {
               sh "make delete" 
            }
            sh "kubectl delete namespace $PREVIEW_NAMESPACE" 
          }
          cleanWs()
        }
   }
}

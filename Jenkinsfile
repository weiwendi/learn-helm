# jenkins
pipeline {
    environment {
	    NAME = 'weiwendi'
	}
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: testAndLint
    image: quay.io/helmpack/chart-testing:v3.0.0-beta.1
    command:
    - sleep
    args:
    - infinity
'''
            // Can also wrap individual steps:
            // container('shell') {
            //     sh 'hostname'
            // }
            defaultContainer 'testAndLint'
        }
    }
	
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
	
    stages {
        stage("Lint") {
            steps {
                
                    sh "ct lint"
                
            }
        }
		
        stage("Install & Test") {
            steps {
                container("testAndLint") {
                    sh "ct install --upgrade"
                }
            }
        }
		stage("Test variable") {
		    steps {
			    echo "${env.NAME}"
			}
		}
    }
}

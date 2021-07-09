pipeline {
    environment {
        githubPagesRepoUrl = 'https://github.com/weiwendi/learn-helm.git'  // Helm Charts 源代码仓库
        githubForkUrl = 'https://github.com/weiwendi/charts.git'  // Helm Charts 包存储库
	}
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: test-and-lint
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
            defaultContainer 'test-and-lint'
        }
    }
	
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
	
    stages {
        stage('Print Messages') {
            steps {
                container("test-and-lint") {
                    echo "workspace: ${WORKSPACE}\n build_id: ${BUILD_ID}"
                    echo "githubPagesRepoUrl: ${githubPagesRepoUrl} branch(gitlab 分支): ${env.BRANCH_NAME}"
                    echo "githubForkUrl: ${githubForkUrl}"
                }
            }
        }	
        stage("Lint") {
            steps {
                sh "ct lint"
            }
        }
    }
}

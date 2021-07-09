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
    image: registry.cn-beijing.aliyuncs.com/sretech/jct:3.4.0
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
                    echo " workspace: ${WORKSPACE}\n githubPagesRepoUrl: ${githubPagesRepoUrl}\n githubForkUrl: ${githubForkUrl}\n branch: ${env.BRANCH_NAME}\n buildId: ${BUILD_ID}"
                }
            }
        }	
        stage("Lint") {
            steps {
                script {
                    sh "ct lint"
                }
            }
        }
        stage("Helm Package") {
            steps {
                script {
                    sh "helm package --dependency-update helm-charts/charts/*"
                    sh "ls -l"
                }
            }
        }
    }
}

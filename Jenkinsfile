// 声明 Pipeline.
pipeline {

    // environment 用于设置环境变量.
    // 这两个环境变量是图表源代码仓库和图表存储库.
    // 你需要把这两个地址修改为你的真实地址.
    environment {

        // Helm Charts 源代码仓库
        githubForkUrl = 'https://github.com/weiwendi/learn-helm.git'

        // Helm Charts 包存储库
        githubPagesRepoUrl = 'https://github.com/weiwendi/charts.git'

        githubAccessKey = credentials('2d511c21-d93e-4f99-ae81-dd631c601922')
    }

    // 定义执行 Job 的 jenkins agent.
    // agent 是运行在 Kubernetes 集群中的一个 Pod. 
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
    # 定义容器的名称
  - name: test-and-release
    # 指定容器的镜像,该镜像包含了 helm/ct/yamllint/yamale/git/kubectl
    image: registry.cn-beijing.aliyuncs.com/sretech/cttools:latest
    command:
    - sleep
    args:
    - infinity
    resources:
      requests:
        memory: "1024Mi"
        cpu: "1000m"
      limits:
        memory: "1024Mi"
        cpu: "1000m"
  nodeSelector:
    public: yes
'''
            // 设置执行各 stages 时默认使用 test-and-release 容器
            defaultContainer 'test-and-release'
        }
    }
	
    // Jenkins 选项
    options {

        // 显示执行的时间，需要 Timestamper 插件支持.
        timestamps()

        // 保留最近 10 次构建记录
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    // stages 设置 pipeline 执行的各个阶段,建议至少包含一个 stage.
    stages {

        // stage 定义了 pipeline 完成的所有实际工作.
        stage('Build Messages') {

            // steps 定义了 pipeline 具体执行的步骤.
            steps {
                
                // 指定使用哪个容器执行该 steps,默认是 test-and-release.
                container("test-and-release") {
                    echo " workspace: ${WORKSPACE}\n githubPagesRepoUrl: ${githubPagesRepoUrl}\n githubForkUrl: ${githubForkUrl}\n branch: ${env.BRANCH_NAME}\n buildId: ${BUILD_ID}"
                }
            }
        }	

        stage("List Changed Charts") {
            steps {
                script {
                    sh "ct list-changed"
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

        stage("Install & Test") {
            steps {
                script {
                    sh "ct install --upgrade"
                }
            }
        }
        stage("Package Charts") {
            steps {
                script {
                    sh "helm package --dependency-update helm-charts/charts/*"
                    sh "ls -l"
                }
            }
        }
        stage("Push Charts to Chart Repo") {
            steps {
                script {
                    def baseBranch = "main"
                    sh "git clone ${env.githubPagesRepoUrl} chart-repo"

                    def repoType
                    if (env.BRANCH_NAME == baseBranch) {
                        repoType = "stable"
                    } else {
                        repoType = "staging"
                    }

                    def files = sh(script: "ls chart-repo", returnStdout: true)
                    if (!files.contains(repoType)) {
                        sh "mkdir chart-repo/${repoType}"
                    }

                    sh "mv ct.yaml chart-repo/${repoType}"

                    sh "helm repo index chart-repo/${repoType}"

                    sh "git config --global user.email 'weiwendi@live.cn'"
                    sh "git config --global user.name 'weiwendi'"

                    dir("chart-repo") {
		        // Add and commit the changes
		        sh "git add --all"
			sh "git commit -m 'pushing charts from branch ${env.BRANCH_NAME}'"
                        withCredentials([usernameColonPassword(credentialsId: '2d511c21-d93e-4f99-ae81-dd631c601922', passwordVariable: 'Password', usernameVariable: 'Username')]) {
			    script {

			        // Inject GitHub auth and push to the repo where charts are being served
				def authRepo = env.githubPagesRepoUrl.replace("://", "://${Username}:${Password}@")
				sh "git push ${authRepo} ${baseBranch}"
			    }
			}
                    }

                }
            }
        }
    }
}

@Library('sharedlibrary') _

pipeline {
    agent any

    environment {
        gitBranchName = "${env.BRANCH_NAME}"
        ecrRegistry = "879381264703.dkr.ecr.ap-south-1.amazonaws.com"
        backendImage = "${ecrRegistry}/backend"
        frontendImage = "${ecrRegistry}/frontend"
        modelImage = "${ecrRegistry}/model"
    }

    stages {
        stage('Init Vars') {
            steps {
                script {
                    env.branchName = env.BRANCH_NAME.replace('/', '-')
                    env.gitCommit = env.GIT_COMMIT ? env.GIT_COMMIT.take(7) : 'latest'
                    env.dockerTag = "${env.branchName}-${env.gitCommit}"
                }
            }
        }

        stage('Git Checkout') {
            steps {
                gitCheckout("${gitRepoURL}", "refs/heads/${gitBranchName}", 'githubCred')
            }
        }

        stage('Docker Build') {
            parallel {
                stage('Build Backend') {
                    steps {
                        dir('movie-analyzer-app/backend') {
                            dockerImageBuild(backendImage, dockerTag)
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        dir('movie-analyzer-app/frontend') {
                            dockerImageBuild(frontendImage, dockerTag)
                        }
                    }
                }
                stage('Build Model') {
                    steps {
                        dir('movie-analyzer-app/model') {
                            dockerImageBuild(modelImage, dockerTag)
                        }
                    }
                }
            }
        }

        stage('Docker Push') {
            parallel {
                stage('Push Backend') {
                    steps {
                        dockerECRImagePush(backendImage, dockerTag, 'ap-south-1')
                    }
                }
                stage('Push Frontend') {
                    steps {
                        dockerECRImagePush(frontendImage, dockerTag, 'ap-south-1')
                    }
                }
                stage('Push Model') {
                    steps {
                        dockerECRImagePush(modelImage, dockerTag, 'ap-south-1')
                    }
                }
            }
        }

        stage('Kubernetes Deploy - DEV') {
            when {
                branch 'dev'
            }
            steps {
                kubernetesEKSHelmDeploy('movie-analyzer', 'dev')
            }
        }

        stage('Kubernetes Deploy - STAGING') {
            when {
                branch 'staging'
            }
            steps {
                kubernetesEKSHelmDeploy('movie-analyzer', 'staging')
            }
        }

        stage('Kubernetes Deploy - PROD') {
            when {
                branch 'main'
            }
            steps {
                kubernetesEKSHelmDeploy('movie-analyzer', 'prod')
            }
        }
    }
}

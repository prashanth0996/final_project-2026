@Library('sharedlibrary')_

pipeline {
    environment {
        gitBranchName = "${env.BRANCH_NAME}"
        ecrRegistry = "879381264703.dkr.ecr.ap-south-1.amazonaws.com"
        backendImage = "${ecrRegistry}/backend"
        frontendImage = "${ecrRegistry}/frontend"
        modelImage = "${ecrRegistry}/model"
        branchName = sh(script: 'echo $BRANCH_NAME | sed "s#/#-#"', returnStdout: true).trim()
        gitCommit = "${GIT_COMMIT[0..6]}"
        dockerTag = "${branchName}-${gitCommit}"
        snykOrg = "14141617-a2e0-4a4f-b558-dce1ea5cad2d"
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    agent  any
    stages {
        stage('Git Checkout') {
            steps {
                gitCheckout("$gitRepoURL", "refs/heads/$gitBranchName", 'githubCred')
            }
        }

        

        stage('Docker Build') {
            parallel {
                stage('Build Backend') {
                    steps {
                        dir('movie-analyzer-app/backend') {
                            dockerImageBuild('$backendImage', '$dockerTag')
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        dir('movie-analyzer-app/frontend') {
                            dockerImageBuild('$frontendImage', '$dockerTag')
                        }
                    }
                }
                stage('Build Model') {
                    steps {
                        dir('movie-analyzer-app/model') {
                            dockerImageBuild('$modelImage', '$dockerTag')
                        }
                    }
                }
            }
        }

        stage('Docker Push') {
            parallel {
                stage('Push Backend') {
                    steps {
                        dockerECRImagePush('$backendImage', '$dockerTag', 'ap-south-1')
                    }
                }
                stage('Push Frontend') {
                    steps {
                        dockerECRImagePush('$frontendImage', '$dockerTag', 'ap-south-1')
                    }
                }
                stage('Push Model') {
                    steps {
                        dockerECRImagePush('$modelImage', '$dockerTag', 'ap-south-1')
                    }
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                dir('helm') {
                    sh """
                        sed -i 's|__BACKEND_IMAGE_REPOSITORY__|${backendImage}|g' values.yaml
                        sed -i 's|__FRONTEND_IMAGE_REPOSITORY__|${frontendImage}|g' values.yaml
                        sed -i 's|__MODEL_IMAGE_REPOSITORY__|${modelImage}|g' values.yaml
                        sed -i 's|__BACKEND_IMAGE_TAG__|${dockerTag}|g' values.yaml
                        sed -i 's|__FRONTEND_IMAGE_TAG__|${dockerTag}|g' values.yaml
                        sed -i 's|__MODEL_IMAGE_TAG__|${dockerTag}|g' values.yaml
                    """
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
                branch 'master'
            }
            steps {
                kubernetesEKSHelmDeploy('movie-analyzer', 'prod')
            }
        }

    }
}

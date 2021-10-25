// USE THE CORRECT LIBRARY TO BUILD FROM. FOR A TAGGED VERSION TO BE BUILD USE THE CORRESPONDING VERSION of shared library.
library "common-code@${'master'}" _

pipeline {
    agent { label 'build_machine' }

    environment {
        REGISTRY = globals.get_docker_internal_artifactory_repository()
        PRODUCTION_REGISTRY = globals.get_docker_production_artifactory_repository()
        GIT_BRANCH = utility.get_git_branch_edited(env.BRANCH_NAME)
        IMAGE_NAME = "${REGISTRY}/csas:${GIT_BRANCH}_${env.BUILD_NUMBER}"
        FINAL_IMAGE = "${REGISTRY}/csas:${GIT_BRANCH}_latest"
        NUM_OLD_BUILDS = globals.get_num_oldbuild()
        ARTIFACTORY_URL = globals.get_artifactory_url()
        SCANNER_HOME = tool 'SonarQubeScanner4'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr:env.NUM_OLD_BUILDS))
    }

    stages {
        stage('CheckNode') {
            steps {
                isUnix()
            }
        }
    	stage('Build') {
            steps {
                script {
                    utility.login_docker_internal_artifactory_repository()
                    writeFile file:"VERSION.txt", text: utility.get_version_string()
                    withCredentials(
                        [usernamePassword(
                                credentialsId: 'artifactory_serviceuser',
                                passwordVariable: 'REPO_PASSWORD',
                                usernameVariable: 'REPO_USERNAME')]) {
                                    sh '''
                                        docker build --build-arg ARTIFACTORY_USERNAME=${REPO_USERNAME} --build-arg ARTIFACTORY_PASSWORD=${REPO_PASSWORD} -t ${IMAGE_NAME} .
                                    '''
                                    try {
                                         utility.login_prisma_console("${IMAGE_NAME}","prisma-image-scan.json") 
                                        prismaCloudPublish resultsFilePattern: 'prisma-image-scan.json' 
                                    }   catch(err){
                                            echo err.getMessage()
                                        }    
                    
                                }
                }
            }
        }
        
        stage('Unit Tests') {
            steps {
                script {
                    sh """
                        echo 'tests need improvements'
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            steps{
                script {
                    try {
                        withSonarQubeEnv('SonarQube') {
                            sh "${SCANNER_HOME}/bin/sonar-scanner -Dproject.settings=./sonar-scanner.properties -Dsonar.branch.name=${env.BRANCH_NAME}"
                        }
                    }   catch (err) {
                            echo err.getMessage()
                        }
                }
            }
        } 
        stage('Push') {
            steps {
                script {
                    utility.login_docker_internal_artifactory_repository()
                    sh """
                        docker push ${IMAGE_NAME}
                        docker tag ${IMAGE_NAME} ${FINAL_IMAGE}
                        docker push ${FINAL_IMAGE}
                    """
                    try {
                         utility.login_prisma_console("${FINAL_IMAGE}","prisma-final-image-scan.json") 
                         prismaCloudPublish resultsFilePattern: 'prisma-final-image-scan.json' 
                    }   catch(err){
                            echo err.getMessage()
                        }
                }
            }
        }
        stage('Deploy Production Image') {
            when {
                expression { return ((env.BRANCH_NAME ==~ /.*-RLS\d*/) || env.BRANCH_NAME ==~ /.*-RC\d+/) }
            }
            steps {
                script {
                    PRODUCTION_IMAGE = "${PRODUCTION_REGISTRY}/csas:${GIT_BRANCH}"
                    utility.login_docker_production_artifactory_repository()
                    sh """
                        docker tag ${IMAGE_NAME} ${PRODUCTION_IMAGE}
                        docker push ${PRODUCTION_IMAGE}
                    """
                    utility.artifactory_create_bundle("${PRODUCTION_IMAGE}","csas")
                    try {
                        utility.login_prisma_console("${PRODUCTION_IMAGE}","prisma-production-image-scan.json") 
                        prismaCloudPublish resultsFilePattern: 'prisma-production-image-scan.json' 
                    }   catch(err){
                            echo err.getMessage()
                        }		                             
                }
            }
        }
    }
    post {

        success {
            sendNotifications 'SUCCESS'
        }
        failure {
            sendNotifications 'FAILED'
            emailext (
                    subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                    recipientProviders: [[$class: 'CulpritsRecipientProvider']]
            )
        }
    }
}

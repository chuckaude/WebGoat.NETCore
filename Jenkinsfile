pipeline {
    agent { label 'galileo' }
    tools { dotnetsdk 'dotnet7-linux64' }
    environment {
        // extract PROJECT_NAME from GIT_URL
        //PROJECT_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
        PROJECT_NAME = 'webgoat-net7'
    }
    stages {
        stage('build') {
            steps {
                sh "dotnet build"
            }
        }
        stage('blackduck') {
            steps {
                withCredentials([string(credentialsId: 'poc329.blackduck.synopsys.com', variable: 'BLACKDUCK_API_TOKEN')]) {
                    sh '''
                        docker build -f Dockerfile.detect -t detect-dotnet .
                        docker run --rm -u $(id -u):$(id -g) -v $WORKSPACE:/source -v $WORKSPACE:/output detect-dotnet \
                            --blackduck.url=$BLACKDUCK_URL --blackduck.api.token=$BLACKDUCK_API_TOKEN \
                            --detect.project.name=$PROJECT_NAME --detect.project.version.name=$BRANCH_NAME --detect.code.location.name=$PROJECT_NAME-$BRANCH_NAME \
                            --detect.policy.check.fail.on.severities=BLOCKER --detect.risk.report.pdf=true
                    '''
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts allowEmptyArchive: true, artifacts: '*_BlackDuck_RiskReport.pdf'
            cleanWs()
        }
    }
}

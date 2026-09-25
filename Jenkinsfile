def COLOR_MAP = [
    'FAILURE': 'danger',
    'SUCCESS': 'good'
]

pipeline{
    agent any
    parameters {
        string(
            name: 'REPO',
            defaultValue: 'cyber0ps'
        )
        choice(
            name: 'BUILD_IMAGE',
            choices: ['auth','converter','gateway','notification'],
            description: 'choose to build'
        )
    }
    stages{
        stage ('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/cyberops-homelab/microservices-python-app.git'
            }
        }
        stage('Docker Build Image'){
            steps{
                sh '''
                    docker build -t ${REPO}/${BUILD_IMAGE}-service:${BUILD_NUMBER} src/${BUILD_IMAGE}-service
                '''
            }
        }
        stage("Docker Push Image"){
            steps{
                withDockerRegistry(url: "https://index.docker.io/v1/", credentialsId: 'docker-hub'){
                    sh "docker push ${REPO}/${BUILD_IMAGE}-service:${BUILD_NUMBER}"
                }
            }
        }
    }
    post {
        always {
            slackSend(
                channel: '#jenkins',
                color: COLOR_MAP[currentBuild.currentResult],
                message: """\
    *${currentBuild.currentResult}:* Job ${env.JOB_NAME}
    Build ${env.BUILD_NUMBER}
    More info at: ${env.BUILD_URL}
    """
            )
        }
    }
}
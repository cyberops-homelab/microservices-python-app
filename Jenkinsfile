def COLOR_MAP = [
    'FAILURE': 'danger',
    'SUCCESS': 'good'
]

pipeline{
    agent any
    parameters {
        string(
            name: 'REPO_DOCKER',
            defaultValue: 'cyber0ps'
        )
        string(
            name: 'GIT_REPO_NAME',
            defaultValue: 'microservices-python-app'
        )     
        string(
            name: 'GIT_USER_NAME',
            defaultValue: 'cyberops-homelab'
        )    
        choice(
            name: 'BUILD_IMAGE',
            choices: ['auth','converter','gateway','notification'],
            description: 'choose to build'
        )
    }
    environment {
        DOCKER_IMAGE = "${REPO_DOCKER}/${BUILD_IMAGE}-service:${BUILD_NUMBER}"
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
                    docker build -t ${DOCKER_IMAGE} src/${BUILD_IMAGE}-service
                '''
            }
        }
        stage("Docker Push Image"){
            steps{
                withDockerRegistry(url: "https://index.docker.io/v1/", credentialsId: 'docker-hub'){
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
        stage('Update Values Image in values.yaml') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "dattnt0209@gmail.com"
                        git config user.name "jenkins"
                        sed -i "s#^image:.*#image: ${DOCKER_IMAGE}#" src/${BUILD_IMAGE}-service/helm_charts/values.yaml
                        git add src/${BUILD_IMAGE}-service/helm_charts/values.yaml
                        git commit -m "Update values image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
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
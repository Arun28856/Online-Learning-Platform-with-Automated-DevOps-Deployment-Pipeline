pipeline {
    agent any

    triggers { githubPush() }

    environment {
        IMAGE     = "lakshmanarun28/learning-platform"
        DOCKERHUB = credentials('dockerhub-creds')
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        INV  = '/home/ubuntu/devops-infra/ansible/inventory.ini'
        KEY  = '/var/lib/jenkins/.ssh/olp_key'
        PLAY = '/home/ubuntu/devops-infra/ansible'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Arun28856/Online-Learning-Platform-with-Automated-DevOps-Deployment-Pipeline.git'
            }
        }

        stage('Validate') {
            steps {
                sh 'test -f index.html'
                sh 'grep -qi "<html" index.html'
                sh 'echo "index.html validation passed"'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE:$BUILD_NUMBER -t $IMAGE:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                sh 'docker push $IMAGE:$BUILD_NUMBER'
                sh 'docker push $IMAGE:latest'
            }
        }

        stage('Deploy to App Servers') {
            steps {
                sh '''
                  ansible-playbook -i $INV --private-key $KEY \
                    -e @$PLAY/group_vars/all.yml \
                    -e image=$IMAGE:$BUILD_NUMBER \
                    $PLAY/deploy.yml
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'ansible app -i $INV --private-key $KEY -m shell -a "docker ps --filter name=learning-app"'
                sh 'ansible app -i $INV --private-key $KEY -m uri -a "url=http://localhost status_code=200"'
            }
        }
    }

    post {
        always  { sh 'docker logout || true' }
        success { echo 'Deployment complete on both app servers' }
        failure { echo 'Pipeline failed, check the stage above' }
    }
}

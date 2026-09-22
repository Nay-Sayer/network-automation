pipeline {

    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    ansible-galaxy collection install \
                        -r requirements.yml
                '''
            }
        }

        stage('Validate') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'network-device-credentials',
                        usernameVariable: 'NETWORK_USERNAME',
                        passwordVariable: 'NETWORK_PASSWORD'
                    )
                ]) {
                    sh '''
                        ansible-playbook \
                            -e vault_username="$NETWORK_USERNAME" \
                            -e vault_password="$NETWORK_PASSWORD" \
                            playbooks/validate.yml
                    '''
                }
            }
        }

        stage('Backup') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'network-device-credentials',
                        usernameVariable: 'NETWORK_USERNAME',
                        passwordVariable: 'NETWORK_PASSWORD'
                    )
                ]) {
                    sh '''
                        ansible-playbook \
                            -e vault_username="$NETWORK_USERNAME" \
                            -e vault_password="$NETWORK_PASSWORD" \
                            playbooks/backup.yml
                    '''
                }
            }
        }
        stage('Approval') {
            steps {
                input message: 'Deploy network configuration?'
            }
        }

        stage('Configure') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'network-device-credentials',
                        usernameVariable: 'NETWORK_USERNAME',
                        passwordVariable: 'NETWORK_PASSWORD'
                    )
                ]) {
                    sh '''
                        ansible-playbook \
                            -e vault_username="$NETWORK_USERNAME" \
                            -e vault_password="$NETWORK_PASSWORD" \
                            playbooks/configure.yml
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'network-device-credentials',
                        usernameVariable: 'NETWORK_USERNAME',
                        passwordVariable: 'NETWORK_PASSWORD'
                    )
                ]) {
                    sh '''
                        ansible-playbook \
                            -e vault_username="$NETWORK_USERNAME" \
                            -e vault_password="$NETWORK_PASSWORD" \
                            playbooks/verify.yml
                    '''
                }
            }
        }
    }
    post {

        always {
            archiveArtifacts artifacts: '*.cfg',
                             allowEmptyArchive: true
        }

        success {
            echo 'Network deployment completed successfully.'
        }

        failure {
            echo 'Network deployment failed.'
        }
    }
}

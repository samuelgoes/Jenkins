pipeline {
    agent { docker { image 'maven:3.3.3' } }
    options {
        skipStagesAfterUnstable()
    }
    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }
    stages {
        stage('Init') {
            steps {
                sh 'printenv'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn --version'

                sh 'echo "Hello World"'

                sh '''
                	echo "Multiline shell steps works too"
                	ls -lah
                '''
            }
        }
        stage('Deploy') {
        	steps {
        		retry(3) {
                    sh './flakey-deploy.sh'
                }

                timeout(time: 3, unit: 'MINUTES') {
                    sh './health-check.sh'
                }

                sh 'echo "Esto aunque pone failed, está OK"'
        	}
        }
        stage('Post-Deploy') {
            steps {
                sh 'echo "Esto sale si el anterior Stage funciona"'
            }
        }
    }
    post {
    	always {
            echo 'This will always run'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'This will run only if successful'
            slackSend channel: '#general',
                  color: 'good',
                  message: "The pipeline ${currentBuild.fullDisplayName} completed successfully."
        }
        failure {
            echo 'This will run only if failed'
            slackSend channel: '#general',
                  color: 'bad',
                  message: "The pipeline ${currentBuild.fullDisplayName} has failed."
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}

pipeline {
    agent any

    environment {
            AWS_DEFAULT_REGION = 'ap-northeast-2'
            S3_BUCKET = 'skdus-build-file'
            JAR_FILE = 'build/libs/apirdsdemo-0.0.1-SNAPSHOT.jar'
            APP_NAME = 'apidemo'
            DEPLOY_GROUP = 'apidemo-group'
            DEPLOY_ZIP = 'deployment.zip'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/leenayeonnn/apidemo', credentialsId: 'leenayeonnn_github_id'
            }
        }
        stage('Build') {
            steps {
                echo 'Building...'
                 sh 'chmod 755 ./gradlew'
                 sh './gradlew build'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // test steps
            }
        }
        stage('Prepare Deployment Package') {
                    steps {
                        echo 'Preparing deployment package...'
                        sh """
                        cd deployment
                        zip -r ${env.DEPLOY_ZIP} appspec.yml scripts/
                        mv ${env.DEPLOY_ZIP} ../
                        cd ..
                        zip -r ${env.DEPLOY_ZIP} -g build/libs/apidemo-0.0.1-SNAPSHOT.jar
                        """
                    }
                }
                stage('Upload to S3') {
                    steps {
                        withAWS(credentials: 'lion-admin') {
                            s3Upload(bucket: env.S3_BUCKET, file: env.DEPLOY_ZIP)
                        }
                    }
                }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                 withAWS(credentials: 'lion-admin') {
                    sh """
                        aws deploy create-deployment \
                        --application-name ${env.APP_NAME} \
                        --deployment-group-name ${env.DEPLOY_GROUP} \
                        --s3-location bucket=${env.S3_BUCKET},key=${env.DEPLOY_ZIP},bundleType=zip
                    """
                 }
            }
        }
    }
}
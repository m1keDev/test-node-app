pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'test-node-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Building commit: ${env.GIT_COMMIT}"
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    SCRIPTS_TEST=$(node -p "try{require('./package.json').scripts.test||''}catch(e){''}" 2>/dev/null || echo "")
                    DEFAULT='echo "Error: no test specified" && exit 1'
                    if [ -z "$SCRIPTS_TEST" ] || [ "$SCRIPTS_TEST" = "$DEFAULT" ]; then
                        echo "No test script configured -- skipping test execution"
                    else
                        npm test
                    fi
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=node_modules/** \
                            -Dsonar.scm.revision=${GIT_COMMIT}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished for commit: ${env.GIT_COMMIT}"
        }
    }
}

pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    echo "===== Environment Information ====="
                    echo "Build Number: ${BUILD_NUMBER}"
                    echo "Branch: ${BRANCH_NAME}"
                    echo "Workspace: ${WORKSPACE}"

                    echo "===== Java ====="
                    java -version || true

                    echo "===== Maven ====="
                    mvn -version || true

                    echo "===== Node.js ====="
                    node --version || true

                    echo "===== npm ====="
                    npm --version || true

                    echo "===== Python ====="
                    python3 --version || true

                    echo "===== pip ====="
                    pip3 --version || true
                '''
            }
        }

        stage('Install Node Dependencies') {
            steps {
                sh '''
                    echo "Installing Node.js dependencies..."

                    for dir in cart catalogue user; do

                        if [ -f "$dir/package.json" ]; then

                            echo "Installing dependencies for $dir"

                            cd "$dir"
                            npm install
                            cd ..

                        fi

                    done
                '''
            }
        }

        stage('Python Validation') {
            steps {
                sh '''
                    echo "Checking Python services..."

                    for dir in payment load-gen; do

                        if [ -f "$dir/requirements.txt" ]; then

                            echo "Found Python service: $dir"

                            python3 -m py_compile "$dir"/*.py || true

                        fi

                    done
                '''
            }
        }

        stage('Java Build') {
            steps {
                sh '''
                    echo "Checking Java service..."

                    if [ -f "shipping/pom.xml" ]; then

                        cd shipping

                        if [ -f "./mvnw" ]; then
                            ./mvnw clean package -DskipTests
                        else
                            mvn clean package -DskipTests
                        fi

                    else

                        echo "No Java Maven project found at expected location."

                    fi
                '''
            }
        }

        stage('Application Tests') {
            steps {
                sh '''
                    echo "Running application tests..."

                    for dir in cart catalogue user; do

                        if [ -f "$dir/package.json" ]; then

                            echo "====================================="
                            echo "Testing $dir"
                            echo "====================================="

                            cd "$dir"

                            if npm run | grep -q "test"; then

                                npm test -- --runInBand || true

                            else

                                echo "No npm test script found in $dir"

                            fi

                            cd ..

                        fi

                    done
                '''
            }
        }
    }
stage('SonarQube Analysis') {
    steps {
        script {
            def scannerHome = tool 'sonarQube Scanner 8.1'

            withSonarQubeEnv('SonarQube') {
                sh """
                    ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=robot-shop \
                    -Dsonar.projectName=robot-shop \
                    -Dsonar.sources=. \
                    -Dsonar.exclusions=**/node_modules/**,**/target/**,**/.git/**
                """
            }
        }
    }
}
    post {

        success {
            echo "Application CI pipeline completed successfully!"
        }

        failure {
            echo "Pipeline failed. Check the Jenkins Console Output."
        }

        always {
            echo "Pipeline execution completed."
        }
    }
}

pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    echo "=== Java ==="
                    java -version || true

                    echo "=== Maven ==="
                    mvn -version || true

                    echo "=== Node ==="
                    node --version || true

                    echo "=== NPM ==="
                    npm --version || true

                    echo "=== Python ==="
                    python3 --version || true

                    echo "=== Pip ==="
                    pip3 --version || true
                '''
            }
        }

        stage('Install Node Dependencies') {
            steps {
                sh '''
                    for dir in cart catalogue user; do
                        if [ -d "$dir" ]; then
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
                    for dir in payment load-gen; do
                        if [ -d "$dir" ]; then
                            echo "Validating Python application: $dir"
                            cd "$dir"

                            echo "Creating Python virtual environment..."
                            python3 -m venv .venv

                            echo "Upgrading pip..."
                            .venv/bin/python -m pip install --upgrade pip

                            if [ -f requirements.txt ]; then
                                echo "Installing Python dependencies..."
                                .venv/bin/pip install -r requirements.txt
                            fi

                            echo "Validating Python files..."
                            .venv/bin/python -m compileall .

                            cd ..
                        fi
                    done
                '''
            }
        }

        stage('Java Build') {
            steps {
                sh '''
                    cd shipping

                    if [ -f ./mvnw ]; then
                        chmod +x ./mvnw
                        ./mvnw clean package -DskipTests
                    else
                        mvn clean package -DskipTests
                    fi

                    echo "Checking compiled Java classes..."

                    if [ -d target/classes ]; then
                        find target/classes -type f | head -20
                    else
                        echo "ERROR: target/classes was not created"
                        exit 1
                    fi

                    cd ..
                '''
            }
        }

        stage('Application Tests') {
            steps {
                sh '''
                    for dir in cart catalogue user; do
                        if [ -d "$dir" ] && [ -f "$dir/package.json" ]; then
                            echo "Running tests for $dir"
                            cd "$dir"

                            npm test -- --runInBand || true

                            cd ..
                        fi
                    done
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=robot-shop \
                            -Dsonar.projectName=robot-shop \
                            -Dsonar.sources=. \
                            -Dsonar.java.binaries=shipping/target/classes \
                            -Dsonar.exclusions=**/node_modules/**,**/target/**,**/.git/**
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs above.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}

pipeline {
    agent any

    environment {
        // We will update this later. For now, this is a good placeholder.
        DOCKER_IMAGE_NAME = "my-juice-shop-app"
        DOCKER_IMAGE_TAG = "build-${env.BUILD_NUMBER}"
    }

    stages {
        stage('1. SAST (Simulation)') {
            steps {
                echo "Simulating SAST Scan on Juice Shop code..."
                sleep 3
                echo "✅ SAST Scan Passed."
            }
        }

        stage('2. Build Docker Image') {
            steps {
                echo "Building Juice Shop Docker image..."
                // This uses the official Dockerfile already in the Juice Shop repo
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
            }
        }

        stage('3. Deploy to Staging') {
            steps {
                echo "Deploying Juice Shop to Staging..."
                // -d = detached, --rm = remove on stop, -p = map port 3000:3000
                sh "docker run -d --rm --name staging-juice-shop -p 3000:3000 ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                echo "Waiting 30 seconds for the application to start..."
                sleep 30 // Give Juice Shop time to initialize
            }
        }

        stage('4. Staging Health Check Gate') {
            steps {
                echo "Verifying Staging is online..."
                // Check if the app is responding on port 3000. If not, fail the pipeline.
                sh 'curl -f http://localhost:3000 || exit 1'
                echo "✅ Health check passed."
            }
        }

        stage('5. DAST (Simulation)') {
            steps {
                echo "Simulating DAST Scan on live staging instance..."
                sleep 5
                echo "✅ DAST Scan Passed."
            }
            post {
                always {
                    echo "Cleaning up Staging environment..."
                    // Stop the staging container regardless of scan result
                    sh "docker stop staging-juice-shop"
                }
            }
        }

        stage('6. Manual Approval Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'All checks passed. Deploy to LIVE?', submitter: 'authorized-users'
                }
            }
        }

        stage('7. Deploy to Live (Simulation)') {
            steps {
                echo "Simulating Production Deployment on port 80..."
                sh "docker run -d --rm --name live-juice-shop -p 80:3000 ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                echo "✅ Juice Shop deployed to Live."
            }
        }

        stage('8. Image Scan (Simulation)') {
            steps {
                echo "Simulating Post-Deploy Image Scan..."
                sleep 3
                echo "✅ Vulnerability Assessment Complete."
            }
        }
    }
}

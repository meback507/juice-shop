// This is the declarative Jenkinsfile for the Security-Driven CI/CD Pipeline project.
// It defines all the stages, from code checkout to simulated production deployment.

pipeline {
    // Run on any available Jenkins agent
    agent any

    environment {
        // Define common variables used throughout the pipeline
        // Using the project name as a prefix is good practice
        DOCKER_IMAGE_NAME = "local-registry/juice-shop"
        DOCKER_IMAGE_TAG  = "build-${env.BUILD_NUMBER}"
        STAGING_HOST      = "staging-juice-shop" // <-- NEW: Use container name as hostname
        APP_PORT          = 3000
        NETWORK_NAME      = "security-pipeline-project_cicd-net" // <-- CHANGED: Use the full network name Docker creates
    }

    stages {
        // This stage is handled automatically by Jenkins when checking out from SCM
        // stage('Checkout') { ... }

        stage('1. Secrets & SAST Scan (Simulation)') {
            steps {
                echo "Simulating Gitleaks (Secrets) and SonarQube (SAST) scans..."
                sleep 3
                echo "✅ Gate Passed: No high-severity issues found."
            }
        }

        stage('2. Build & Push Docker Image') {
            steps {
                echo "Building Juice Shop Docker image..."
                // Build the image using the official Dockerfile from the repo
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."

                echo "Pushing image to local registry..."
                sh "docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} localhost:5000/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                sh "docker push localhost:5000/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
            }
        }

        stage('3. Deploy to Staging') {
            steps {
                echo "Deploying Juice Shop to Staging environment..."
                // Key Changes:
                // 1. --network: Connects the container to our shared network.
                // 2. --name: Gives the container a predictable hostname (STAGING_HOST).
                sh """
                    docker run -d --rm \
                      --network ${NETWORK_NAME} \
                      --name ${STAGING_HOST} \
                      -p ${APP_PORT}:${APP_PORT} \
                      localhost:5000/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                """ // <-- CHANGED: Deploys using the image from our local registry and attaches to network

                echo "Waiting 30 seconds for the application to start..."
                sleep 30 // Give Juice Shop time to initialize fully
            }
        }

        stage('4. Staging Health Check Gate') {
            steps {
                echo "Verifying Staging instance is online..."
                // Key Change:
                // We now curl the container by its name on the Docker network, not localhost.
                sh "curl -f http://${STAGING_HOST}:${APP_PORT} || exit 1" // <-- CHANGED
                echo "✅ Health Check Passed."
            }
        }

        stage('5. DAST Scan (Simulation)') {
            steps {
                echo "Simulating OWASP ZAP DAST scan on http://${STAGING_HOST}:${APP_PORT}..."
                sleep 5
                echo "✅ DAST Scan Passed: No critical vulnerabilities found."
            }
            post {
                // This 'always' block ensures cleanup happens even if the stage fails
                always {
                    echo "Cleaning up Staging environment..."
                    // Stop the staging container by its name
                    sh "docker stop ${STAGING_HOST}"
                }
            }
        }

        stage('6. Manual Approval Gate') {
            steps {
                // This pauses the pipeline and waits for a user to approve.
                // It will automatically abort after 5 minutes if there is no input.
                timeout(time: 5, unit: 'MINUTES') {
                    input message: "All automated checks passed. Ready to deploy to LIVE?", submitter: 'admin' // Use your Jenkins username
                }
            }
        }

        stage('7. Deploy to Live (Simulation)') {
            steps {
                echo "Simulating Production Deployment on port 80..."
                // For a real deployment, this might be a different cluster or server.
                // Here, we just map to a different host port.
                sh """
                    docker run -d --rm \
                      --name live-juice-shop \
                      -p 80:3000 \
                      localhost:5000/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                """
                echo "✅ Juice Shop deployed to Live at http://localhost:80"
            }
        }

        stage('8. Image Scan & Report (Simulation)') {
            steps {
                echo "Simulating Trivy post-deployment image scan..."
                sleep 3
                echo "✅ Vulnerability Assessment Complete."
            }
            post {
                always {
                    echo "Cleaning up Live environment simulation..."
                    sh "docker stop live-juice-shop || true" // Use '|| true' to prevent failure if container is already stopped
                }
            }
        }
    }
}

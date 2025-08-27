pipeline {
    agent any

    stages {
        stage('Build Artifact') {
            steps {
                // Build the project and skip tests
                sh "mvn clean package -DskipTests=true"
                // Archive the built JAR file for later access
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Unit Test') {
            steps {
                // Run unit tests
                sh "mvn test"
            }
            post {
                always {
                    // Publish JUnit test results
                    junit 'target/surefire-reports/*.xml'
                    // Publish JaCoCo code coverage report
                    jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }

        stage('Mutation Tests - PIT') {
            steps {
                // Run mutation tests using PIT
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
                always {
                    // Publish PIT mutation test results
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def mvn = tool 'Default Maven' // Ensure Maven is installed and configured
                    withSonarQubeEnv() {
                        sh "${mvn}/bin/mvn clean verify sonar:sonar " +
                           "-Dsonar.projectKey=num-app " +
                           "-Dsonar.projectName='num-app' " +
                           "-Dsonar.host.url=http://192.168.10.15:9000 " +
                           "-Dsonar.token=sqp_0ee5e69137dbdcb58b3b31600ee2bde9ab4003e6"
                    }
                }
            }
        }

        stage('Kubernetes Deployment - DEV') {
            steps {
                // Use Kubernetes configuration with specified credentials
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    script {
                        try {
                            // Update the deployment YAML with the new image tag
                            sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                            // Apply the Kubernetes deployment configuration
                            sh "kubectl apply -f k8s_deployment_service.yaml"
                            // Check deployment status
                            sh "kubectl rollout status deployment/devsecops"
                        } catch (Exception e) {
                            error "Deployment failed: ${e.message}"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace after the build
            cleanWs()
        }
        success {
            // Notify on success (e.g., send an email or Slack message)
            echo "Pipeline completed successfully!"
        }
        failure {
            // Notify on failure (e.g., send an email or Slack message)
            echo "Pipeline failed. Please check the logs."
        }
    }
}

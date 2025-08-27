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
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
            post {
                always {
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }
        

        stage('Kubernetes Deployment - DEV') {
            steps {
                // Use Kubernetes configuration with specified credentials
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    // Update the deployment YAML with the new image tag
                    sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                    // Apply the Kubernetes deployment configuration
                    sh "kubectl apply -f k8s_deployment_service.yaml"
                }
            }

        }
    }
}

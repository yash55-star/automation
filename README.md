## devops-automation



## GIT+Maven+docker image creation + docker image run



pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven_3_5_0"
    }

    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'main', url: 'https://github.com/yash55-star/automation.git'

                // Run Maven on a Windows agent.
                bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                bat "docker build -t yash1107/devops-integration ."
            }
        }

        stage('Run Docker Image') {
            steps {
                // Run the Docker image and expose port 8081, wait for it to finish for 15 minutes
                bat "start /B docker run -p 8081:8081 yash1107/devops-integration"
                timeout(time: 15, unit: 'MINUTES') {
                    // Wait for the Docker container to finish
                    bat 'timeout /t 900'
                }
            }
        }
    }
}


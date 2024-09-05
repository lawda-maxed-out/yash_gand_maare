pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-credentials'
        NEXUS_CREDENTIALS_ID = 'nexus'
        SONARQUBE_CREDENTIALS_ID = 'sonar'
        SONARQUBE_URL = 'http://localhost:9000/'
        NEXUS_URL = 'http://localhost:8081/'
        NEXUS_REPOSITORY = 'http://localhost:8081/repository/nexus-repo-ashu/'
    }

    // triggers {
    //     // Commented out since GitHub webhook is not used
    //     // githubPush()
    // }
    tools {
        maven 'Maven 3.8.7' // Use the Maven tool configured in Jenkins
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/lawda-maxed-out/yash_gand_maare.git'
            }
        }

        stage('Build') {
            steps {
                dir('maven-app/my-app') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=your-project-key -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_CREDENTIALS_ID'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactId = pom.artifactId
                    def version = pom.version
                    def packaging = pom.packaging
                    def file = "target/${artifactId}-${version}.${packaging}"
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "$NEXUS_URL",
                        groupId: pom.groupId,
                        version: version,
                        repository: "$NEXUS_REPOSITORY",
                        credentialsId: "$NEXUS_CREDENTIALS_ID",
                        artifacts: [
                            [artifactId: artifactId, classifier: '', file: file, type: packaging]
                        ]
                    )
                }
            }
        }
    }

    post {
        always {
            emailext(
                to: 'pandaashutosh350@gmail.com',
                subject: "Jenkins Build ${currentBuild.fullDisplayName}",
                body: "Build ${currentBuild.fullDisplayName} completed with status: ${currentBuild.result}"
            )
        }
    }
}
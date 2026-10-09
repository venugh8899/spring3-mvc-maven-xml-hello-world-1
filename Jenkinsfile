pipeline {
    agent any

    tools {
        // Must match Maven installation name in Jenkins
        maven "Maven3"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.133.145.136:8081"
        NEXUS_REPOSITORY = "devops"
        NEXUS_CREDENTIAL_ID = "nexus-hiring-credentials"
    }

    stages {

        stage("Clone Code") {
            steps {
                echo "Cloning source code from GitHub"

                git 'https://github.com/venugh8899/spring3-mvc-maven-xml-hello-world-1.git'
            }
        }

        stage("Maven Build") {
            steps {
                echo "Building application using Maven"

                sh 'mvn -B -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    // Read Maven project metadata
                    def pom = readMavenPom file: "pom.xml"

                    // Avoid the restricted pom.packaging getter
                    def packaging = sh(
                        script: 'mvn -q help:evaluate -Dexpression=project.packaging -DforceStdout',
                        returnStdout: true
                    ).trim()

                    def groupId = sh(
                        script: 'mvn -q help:evaluate -Dexpression=project.groupId -DforceStdout',
                        returnStdout: true
                    ).trim()

                    def artifactId = sh(
                        script: 'mvn -q help:evaluate -Dexpression=project.artifactId -DforceStdout',
                        returnStdout: true
                    ).trim()

                    echo "Group ID: ${groupId}"
                    echo "Artifact ID: ${artifactId}"
                    echo "Packaging: ${packaging}"
                    echo "Build Number: ${BUILD_NUMBER}"

                    // Locate the generated artifact
                    def filesByGlob = findFiles(
                        glob: "target/*.${packaging}"
                    )

                    if (filesByGlob.length == 0) {
                        error "No ${packaging} artifact found in target directory"
                    }

                    def artifactPath = filesByGlob[0].path

                    if (!fileExists(artifactPath)) {
                        error "Artifact not found: ${artifactPath}"
                    }

                    echo "Artifact found: ${artifactPath}"

                    // Upload artifact and POM to Nexus
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: groupId,
                        version: "${BUILD_NUMBER}",
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [
                                artifactId: artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: packaging
                            ],
                            [
                                artifactId: artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"
                            ]
                        ]
                    )

                    echo "Artifact uploaded to Nexus successfully!"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }

        failure {
            echo "Pipeline failed. Check the failed stage logs."
        }

        always {
            echo "Pipeline execution finished."
        }
    }
}

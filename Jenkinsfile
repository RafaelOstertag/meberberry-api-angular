pipeline {
    agent {
        label 'java&&nodejs'
    }

    tools {
        maven 'Latest Maven'
    }

    parameters {
        string defaultValue: 'none', description: 'Version to deploy', name: 'VERSION', trim: true
    }

    options {
        ansiColor('xterm')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '15')
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Build Package and deploy') {
            when {
                allOf {
                    expression { return params.VERSION != 'none' }
                    expression { return params.VERSION != '' }
                }
            }

            environment {
                NPM_VERSION = sh returnStdout: true, script: 'echo $VERSION | sed -e "s/-SNAPSHOT\\$/-dev.$BUILD_NUMBER/"'
            }

            steps {
                configFileProvider([configFile(fileId: 'b958fc4b-b1bd-4233-8692-c4a26a51c0f4', variable: 'MAVEN_SETTINGS_XML')]) {
                    sh 'mvn -B -s "$MAVEN_SETTINGS_XML" -Dmemberberry-api.version=${VERSION} -Dpackage.version=${NPM_VERSION} clean package'
                }
                configFileProvider([configFile(fileId: 'a7bcaf1c-8a86-4633-bc22-2755c797b0fa', targetLocation: 'target/memberberry-server-api/.npmrc')]) {
                    sh 'mvn -B exec:exec@publish-memberberry-server-angular-package'
                }
            }
        }
    }

    post {
        unsuccessful {
            mail to: "rafi@guengel.ch",
                    subject: "${JOB_NAME} (${BRANCH_NAME};${env.BUILD_DISPLAY_NAME}) -- ${currentBuild.currentResult}",
                    body: "Refer to ${currentBuild.absoluteUrl}"
        }
    }
}

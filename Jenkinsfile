pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'LocalBranch', localBranch: 'master']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'f6a9e767-b103-4249-b04f-dca92e758936', name: 'origin', url: 'https://github.com/vitaliymashkov/chat.git']]])
            }
        }

        stage('Set build num') {
            steps {
                sh 'npm install'
                sh "npm version 0.1.${env.BUILD_ID}"
                sh "echo 'export const VERSION = \"1.5.${env.BUILD_ID}\";' > 'src/version.ts'"
            }
        }


        stage('Make CHANGELOG') {
            steps {
                sh 'chmod 777 ./make_changelog.sh'
                sh 'chmod 777 ./CHANGELOG.md'
                sh "./make_changelog.sh 0.1.${env.BUILD_ID} `head -n 1 CHANGELOG.md | awk '{print \$2}'`"
            }
        }

        stage('Commit') {
            environment {
                GITUSER = credentials('f6a9e767-b103-4249-b04f-dca92e758936')
            }
            steps {
                sh 'git add .'
                sh "git commit -m \"update version to v0.1.${env.BUILD_ID}\""
                sh "git push https://${GITUSER_USR}:${GITUSER_PSW}@github.com/vitaliymashkov/chat.git origin v0.1.${env.BUILD_ID}"
            }
        }

        stage('Push') {
            environment {
              GITUSER = credentials('f6a9e767-b103-4249-b04f-dca92e758936')
            }
            steps {
                sh "git push https://${GITUSER_USR}:${GITUSER_PSW}@github.com/vitaliymashkov/chat.git origin master"
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                echo "Tests not defined"
            }
        }

        stage('Deploy') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'chat', transfers: [sshTransfer(excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: 'public/', sourceFiles: 'public/**/*')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
}
#!groovy

@Library('github.com/ayudadigital/jenkins-pipeline-library@v6.2.0') _

// Initialize global config
cfg = jplConfig('gp-sonarqube', 'bash', '', [email: env.CI_NOTIFY_EMAIL_TARGETS])

/**
 * Build and publish docker images
 *
 * @param nextReleaseNumber String Release number to be used as tag
 */
def buildAndPublishDockerImage(nextReleaseNumber = "") {
    if (nextReleaseNumber == "") {
        nextReleaseNumber = jplGetNextReleaseNumber(cfg).substring(1)
    }
    docker.withRegistry("", 'docker-token') {
        def customImage = docker.build("${env.DOCKER_ORGANIZATION}/${cfg.projectName}:${nextReleaseNumber}", "--pull --no-cache ${cfg.projectName.substring(3)}")
        customImage.push()
        if (nextReleaseNumber != "beta") {
            customImage.push('latest')
        }
    }
}

pipeline {
    agent { label 'docker' }

    stages {
        stage ('Initialize') {
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Bash linter') {
            steps {
                sh 'devcontrol run-bash-linter'
            }
        }
        stage ('Build') {
            steps {
                buildAndPublishDockerImage("beta")
            }
        }
        stage ('Make release') {
            when { branch 'release/new' }
            steps {
                buildAndPublishDockerImage()
                jplMakeRelease(cfg, true)
                deleteDir()
            }
        }
    }

    post {
        always {
            jplPostBuild(cfg)
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(artifactNumToKeepStr: '20',artifactDaysToKeepStr: '30'))
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }
}

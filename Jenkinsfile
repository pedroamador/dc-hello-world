#!groovy

@Library('github.com/red-panda-ci/jenkins-pipeline-library@v2.7.0') _

// Initialize global config
cfg = jplConfig('dc-hello-world', 'bash', '', [slack: '#integrations', email:'redpandaci+dc-hello-world@gmail.com'])

pipeline {
    agent none

    stages {
        stage ('Initialize') {
            agent { label 'master' }
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Build') {
            agent { label 'docker' }
            steps {
                script {
                    jplDockerPush (cfg, "kairops/dc-hello-world", "test", ".", "https://registry.hub.docker.com", "cikairos-docker-credentials")
                }
            }
        }
        stage ('Test') {
            agent { label 'docker' }
            steps  {
                sh 'bin/test.sh'
            }
        }
        stage ('Release confirm') {
            when { expression { cfg.BRANCH_NAME.startsWith('release/v') || cfg.BRANCH_NAME.startsWith('hotfix/v') } }
            steps {
                jplPromoteBuild(cfg)
            }
        }
        stage ('Release finish') {
            agent { label 'master' }
            when { expression { (cfg.BRANCH_NAME.startsWith('release/v') || cfg.BRANCH_NAME.startsWith('hotfix/v')) && cfg.promoteBuild.enabled } }
            steps {
                jplDockerPush (cfg, "kairops/dc-hello-world", cfg.releaseTag, ".", "https://registry.hub.docker.com", "cikairos-docker-credentials")
                jplDockerPush (cfg, "kairops/dc-hello-world", "latest", ".", "https://registry.hub.docker.com", "cikairos-docker-credentials")
                jplCloseRelease(cfg)
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
        skipDefaultCheckout()
        timeout(time: 1, unit: 'DAYS')
    }
}
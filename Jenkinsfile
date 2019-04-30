@Library("global-pipeline-libraries@v1.8.3") _

pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
        ansiColor('xterm')
    }

    triggers {
        issueCommentTrigger('.*test this please.*')
    }

    stages {
        stage('Checkout tags') {
            // Get tags, because default checkout does not include them.
            steps {
                checkout scm: [
                        $class           : "GitSCM",
                        branches         : scm.branches,
                        extensions       : [[
                                                    $class   : "CloneOption",
                                                    noTags   : false,
                                                    reference: "",
                                                    shallow  : false]],
                        userRemoteConfigs: scm.userRemoteConfigs
                ]
            }
        }

        stage('Build, Test, Publish') {
            agent {
                docker {
                    image "docker.werally.in/java8-sbt:0.13.8"
                    reuseNode true
                    label 'ec2'
                    args '-v /etc/passwd:/etc/passwd:ro'
                }
            }
            environment {
                HOME = "${env.WORKSPACE}"
            }
            steps {
                rally_sbt(["user.home": "${env.HOME}"], "+test +publish")
            }
        }
    }
}

#!/bin/env groovy

pipeline {

  agent none


  stages {
    stage('Init') {
      agent {
        docker {
          image 'eclipse-temurin:8-jdk'
        }
      }
      environment {
        GIT_BRANCH_NAME = sh (script: 'git name-rev --name-only HEAD', returnStdout: true).trim().minus(~/^remotes\/origin\//)
        IS_SNAPSHOT = getMavenVersion().endsWith("-SNAPSHOT")
        MAVEN_CONFIG = " -Dform-dist-repo.snapshots.url=${params.MAVEN_SNAPSHOTS_REPO}"
      }
      steps {
        script {
          echo "init"
        }
      }
    }
    stage('Build') {
      steps {
        script {
          tags_extra = GIT_BRANCH_NAME ==~ /develop/ ? 'develop' : ''
        }
        sh "./mvnw -B  clean package -DskipTests"
        archive includes: '**/target/*.jar'
        stash includes: '**/target/*.jar', name: 'jar'
      }
    }
    stage('Unit test') {
      steps {
        unstash 'jar'
        sh './mvnw -B test'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: "**/surefire-reports/**/*.xml"
        }
      }
    }
    stage('Integration test') {
      steps {
        unstash 'jar'
        sh './mvnw -B -Ddocker.skip verify'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults:"**/failsafe-reports/**/*.xml"
        }
      }
    }
    stage('Static Analysis') {
      steps {
        // TODO: run static analyses pmt, findbugs, etc
        echo "Done"
      }
    }
    stage('Functional tests') {
      steps {
        // TODO: run functional tests from image (docker-compose)
        sh 'echo Functional test'
      }
    }
  }
}

def getMavenVersion() {
  def version = sh(script: "./mvnw org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate -Dexpression=project.version | grep -v '\\['  | tail -1", returnStdout: true).trim()
  println version
  return version
}

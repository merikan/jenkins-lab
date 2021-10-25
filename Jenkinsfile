#!/bin/env groovy

pipeline {

  agent none


  stages {
    stage('Init') {
      agent {
        dockerfile {
          dir '.jenkins'
          filename 'Dockerfile-init'
        }
      }
      steps {
        script {
          env.GIT_BRANCH_NAME = sh (script: 'git name-rev --name-only HEAD', returnStdout: true).trim().minus(~/^remotes\/origin\//)
          env.IS_SNAPSHOT = getMavenVersion().endsWith("-SNAPSHOT")
          env.MAVEN_CONFIG = " -Dform-dist-repo.snapshots.url=${params.MAVEN_SNAPSHOTS_REPO}"
        }
      }
    }
    stage('Build') {
      agent {
        docker {
          image 'eclipse-temurin:17-jdk'
        }
      }
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
      agent {
        docker {
          image 'eclipse-temurin:17-jdk'
        }
      }
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
      agent {
        docker {
          image 'eclipse-temurin:17-jdk'
          args '--volume /var/run/docker.sock:/var/run/docker.sock'
        }
      }
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
      agent {
        docker {
          image 'eclipse-temurin:17-jdk'
        }
      }
      steps {
        // TODO: run static analyses pmt, findbugs, etc
        echo "Done"
      }
    }
    stage('Functional tests') {
      agent {
        docker {
          image 'eclipse-temurin:17-jdk'
        }
      }
      steps {
        // TODO: run functional tests
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

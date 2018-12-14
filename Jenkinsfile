#!groovy
env.gitToken = "0407d5b5693b40bb39b8e0db51ac778111356139"

import groovy.json.JsonSlurperClassic
  echo env.BUILD_NUMBER
  env.PATH = "/Users/admin/.rvm/gems/ruby-2.4.1/bin:/Users/admin/.rvm/gems/ruby-2.4.1@global/bin:/Users/admin/.rvm/rubies/ruby-2.4.1/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/admin/.rvm/bin"
  sh "source ~/.bash_profile"
  currentBuild.result = "SUCCESS"

    echo "+++branch: ${env.branch}"
    echo "+++upload: ${env.upload}"
    echo "+++commit: ${env.commit}"
    echo "+++lane: ${env.lane}"
    echo "+++ statusUrl: ${env.statusUrl}"
    echo "+++ commentUrl: ${env.commentUrl}"



    if (env.lane == "appstore") {
      try {
        deployToAppStore()
      } catch (e) {
        currentBuild.result = "FAILURE"
        return
      }
    } else if (env.upload == "true") {
      try {
        buildStarted()
        deployBeta(env.branch)
      } catch (e) {
        currentBuild.result = "FAILURE"
        buildFinal()
        return
      }
    } else {
      try {
        buildStarted()
        runUnitTests("", env.branch)
      } catch (e) {
        currentBuild.result = "FAILURE"
        buildFinal()
        return
      }
    }

    buildFinal()

    def runUnitTests(lane, branch) {
      checkout(lane, branch)
      cleanEnvironment()
      cocoapods()
      // checkStyle()
      unitTests()
    }

    def cleanEnvironment() {
      stage('Prepare environment') {
        parallel(
          reset_simulators: {
            prepareEnvironment()
          },
          clean_folders: {
            cleanFolders()
          }
        )
      }
    }

    def unitTests() {
      stage('UnitTests') {
        sh 'fastlane ios lassie_run_tests'
      }
    }

    def cocoapods() {
      stage('cocoapods') {
        sh 'fastlane install_intelligent_carthage'
      }
    }

    def checkStyle() {
      stage('check style') {
        sh 'fastlane ios check_style_pr'
      }
    }

    def prepareEnvironment() {
        KEYCHAIN="/Users/ios_slave/Library/Keychains/login.keychain-db"
        sh "security -v list-keychains -s ${KEYCHAIN}"
        sh "security -v unlock-keychain -p mobile-ci ${KEYCHAIN}"
        sh "security set-keychain-settings -t 3600 -l ${KEYCHAIN}"

        sh "xcrun simctl erase all"
    }

    def cleanFolders() {
        sh "rm -rf /Users/ios_slave/workspace/ios-n-apps/bluepill/"
        sh "rm -rf /Users/ios_slave/workspace/ios-n-apps/output/"
        sh "rm -rf /Users/ios_slave/workspace/ios-n-apps/derivedData/"
    }

    def checkout(lane, branch) {
      stage('Checkout') {
        def branchToCheckout = ""
        if (lane == "appstore") {
          branchToCheckout = "origin/master"
        } else {
          branchToCheckout = "origin/feature/${branch}"
        }
        checkout([
                $class                           : 'GitSCM',
                branches                         : [[name: branchToCheckout]],
                doGenerateSubmoduleConfigurations: false,
                extensions                       : [],
                submoduleCfg                     : [],
                userRemoteConfigs                : [
                        [credentialsId: 'ios-ci',
                         url          : 'https://github.com/doghero/lassie.git']
                ]
        ])
      }
    }

    def changePrStatus(state, message) {
        if (env.statusUrl.length() > 0) {
            prStatus([
                    authToken  : env.gitToken,
                    statusesUrl: env.statusUrl,
                    prStatus   : state,
                    description: message,
                    prContext  : "Ci/Jenkins",
                    targetUrl  : env.BUILD_URL
            ])
        }
    }

    def commentGithub(commentMessage) {
        if (env.commentUrl.length() > 0) {
            gitComment([
                    authToken : env.gitToken,
                    commentUrl: env.commentUrl,
                    message   : commentMessage
            ])
        }
    }

    def buildStarted() {
      changePrStatus("pending", "In progress...")
      //updateStatus("pending", "InProgress...")
    }

    def buildSuccess() {
      changePrStatus("success", "The build succeeded!")
    }

    def buildError() {
      changePrStatus("failure", "Build failed")
    }

    def buildFinal() {
      echo currentBuild.result
      if (currentBuild.result == 'SUCCESS') {
          buildSuccess()
          commentGithub("Coverage: "+ getCoverage())
          // slackSend channel: '#mobile-monitor-ios', color: "#339c4a", message: "Build: ${env.JOB_NAME} ${env.branch} (#${env.BUILD_NUMBER}) finalizado com sucesso(<${env.BUILD_URL}|Open>)"
      } else {
          buildError()
          // slackSend channel: '#mobile-monitor-ios', color: "#ff0000", message: "ended ${env.JOB_NAME} ${env.branch} (#${env.BUILD_NUMBER}) with failure (<${env.BUILD_URL}|Open>)"
      }
    }

    def getCoverage() {
      coverage = sh(
            script: "cat '/Users/ios_slave/workspace/lassie/jenkins_build/code-coverage/report.json' | jq '.coverage'",
            returnStdout: true
            )
      return coverage
    }

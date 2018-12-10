#!groovy
env.gitToken = "0cce0beac62fe27550c4cb697e3cd121d0eac4f2"

import groovy.json.JsonSlurperClassic
node {
  echo env.BUILD_NUMBER
  env.PATH = "/Users/admin/.rvm/gems/ruby-2.4.1/bin:/Users/admin/.rvm/gems/ruby-2.4.1@global/bin:/Users/admin/.rvm/rubies/ruby-2.4.1/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/admin/.rvm/bin"

  currentBuild.result = "SUCCESS"

    echo "+++branch: ${env.branch}"
    echo "+++upload: ${env.upload}"
    echo "+++commit: ${env.commit}"
    echo "+++lane: ${env.lane}"
    echo "+++ statusUrl: ${env.statusUrl}"
    echo "+++ commentUrl: ${env.commentUrl}"

    sh "source ~/.bash_profile"

    // if (env.lane == "appstore") {
    //   try {
    //     deployToAppStore()
    //   } catch (e){
    //     currentBuild.result = "FAILURE"
    //     return
    //   }
    // } else if (env.upload == "true") {
    //   try {
    //     buildStarted()
    //     deployBeta(env.branch)
    //   } catch (e) {
    //     currentBuild.result = "FAILURE"
    //     buildFinal()
    //     return
    //   }
    // } else {
    //   try {
    //     buildStarted()
    //     runUnitTests("", env.branch)
    //   } catch (e) {
    //     currentBuild.result = "FAILURE"
    //     buildFinal()
    //     return
    //   }
    // }
    //
    // buildFinal()
  }

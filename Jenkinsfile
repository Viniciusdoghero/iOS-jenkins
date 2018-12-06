#!groovy
node() {
  env.PATH = "/Users/admin/.rvm/gems/ruby-2.4.1/bin:/Users/admin/.rvm/gems/ruby-2.4.1@global/bin:/Users/admin/.rvm/rubies/ruby-2.4.1/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/admin/.rvm/bin"
    def PR_APPROVED = "approved"
    def LB_QA = "QA"
    def SYNCHRONIZE = "synchronize"
    def OPENED = "opened"
    def TRUE = "true"
    def FALSE = "false"
    //get the name of the repository to know the job that needs to be executed
    repoName = sh(
            script: "echo '${env.payload}' | jq '.repository.name' | sed -e 's/\"//g'",
            returnStdout: true
    ).trim()
    //set the build name to know if is running Android or iOS
    try {
        if (repoName != null && repoName.length() > 0) {
            currentBuild.displayName = repoName
        }
    } catch(error) {
        echo "+++Setting build name error: " + error
    }
    //if the job is called without the payload ignore everything(that must never happend)
    if (env.payload != null && env.payload.length() > 2) {
        echo "+++payload is not empty"
        //check if the payload have the pull_request object
        prObj = sh(
                script: "echo '${env.payload}' | jq '.pull_request'",
                returnStdout: true
        ).trim()

        if (prObj.trim().length() > 2) {
            echo "+++pull_request is not empty"
            prUrl = sh(
                    script: "echo '${env.payload}' | jq '.pull_request.issue_url'",
                    returnStdout: true
            ).trim()

            commentUrl = sh(
                    script: "echo '${env.payload}' | jq '.pull_request._links.comments.href'",
                    returnStdout: true
            ).trim()

            statusUrl = sh(
                    script: "echo '${env.payload}' | jq '.pull_request._links.statuses.href'",
                    returnStdout: true
            ).trim()

            branch = sh(
                    script: "echo '${env.payload}' | jq '.pull_request.head.ref' | sed -e 's/\"//g' | cut -d ' ' -f2- | cut -d '/' -f2-",
                    returnStdout: true
            ).trim()

            action = sh(
                    script: "echo '${env.payload}' | jq '.action' | sed -e 's/\"//g'",
                    returnStdout: true
            ).trim()

            prApproved = sh(
                    script: "echo '${env.payload}' | jq '.review.state' | sed -e 's/\"//g'",
                    returnStdout: true
            ).trim()
            commitId = sh(
                    script: "echo '${env.payload}' | jq '.pull_request.head.sha' | sed -e 's/\"//g'",
                    returnStdout: true
            ).trim()

            //if action is synchronize, means that the trigger came from simple push on a PR, so no need to upload
            if (action == SYNCHRONIZE || action == OPENED) {
                echo "+++starting job from push (action = ${action})"
                startJob (repoName, checkQaLabel(prUrl), commitId, branch, commentUrl, statusUrl)
            } else if (prApproved == PR_APPROVED) {
                //if action is not synchronize and state is approved, means that need to upload
                echo "+++starting job from PR comment (${action} != ${SYNCHRONIZE} && ${OPENED})"
                startJob (repoName, TRUE, commitId, branch, commentUrl, statusUrl)
            }
        } else {
            //is case pull_requestm does not exists, just run the job whitout upload
            echo "+++pull_request is empty"
            branch = sh(
                    script: "echo '${env.payload}' | jq '.ref' | sed -e 's/\"//g' | cut -d '/' -f4-",
                    returnStdout: true
            ).trim()

            commitId = sh(
                    script: "echo '${env.payload}' | jq '.before' | sed -e 's/\"//g'",
                    returnStdout: true
            ).trim()
            echo "+++starting job from push"
            startJob (repoName, FALSE, commitId, branch, "", "")
        }
    }
}

def checkQaLabel(prUrl) {
    def LB_QA = "QA"
    def TRUE = "true"
    def FALSE = "false"

    try {
        echo "+++Chacking PR label: ${prUrl}"
        label = sh(
            script: "curl -H 'Authorization: token 0641455876e8e575a63fc26a975424ae08c32297 ${prUrl}/labels | jq '.[] | select(.name == \"${LB_QA}\")'",
            returnStdout: true
        ).trim()

        echo "+++label: ${label}"
        if (label.length() > 0) {
            echo "+++Chacking PR label: TRUE"
            return TRUE
        } else {
            echo "+++Chacking PR label: FALSE"
            return FALSE
        }
    } catch(error) {
        echo "+++Chacking PR label: FALSE:catch"
        return FALSE
    }
}

def startJob( jobName , upload, commit, branch, commentUrl, statusUrl ) {
    echo "+++jobName: ${jobName}"
    echo "+++upload: ${upload}"
    echo "+++commit: ${commit}"
    echo "+++branch: ${branch}"
    echo "+++commentUrl: ${commentUrl}"
    echo "+++statusUrl: ${statusUrl}"

    stage ('Starting job') {
        build job: jobName, parameters: [
            [$class: 'StringParameterValue', name: 'upload', value: upload],
            [$class: 'StringParameterValue', name: 'commit', value: commit],
            [$class: 'StringParameterValue', name: 'branch', value: branch],
            [$class: 'StringParameterValue', name: 'commentUrl', value: commentUrl],
            [$class: 'StringParameterValue', name: 'statusUrl', value: statusUrl]
        ]
    }
}

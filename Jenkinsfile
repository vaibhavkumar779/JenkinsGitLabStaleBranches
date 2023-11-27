pipeline {
    agent any

    parameters {
        string(name: 'staleBranch', defaultValue: 'feature/stale', description: 'Specify the stale branch name')
        int(name: 'staleThreshold', defaultValue: 90, description: 'Specify the staleness threshold in days')
    }

    stages {
        stage('List Stale Branches') {
            steps {
                script {
                    def currentTimestamp = new Date().toInstant().toEpochMilli()
                    def staleBranches = []

                    sh(script: 'git ls-remote --heads origin', returnStdout: true).eachLine { branchLine ->
                        def branchInfo = branchLine.split()
                        def branchName = branchInfo[1].replaceAll(/.*refs\/heads\//, '').trim()
                        def lastCommitTimestamp = branchInfo[0].toLong() * 1000

                        def stalenessInDays = (currentTimestamp - lastCommitTimestamp) / (24 * 60 * 60 * 1000)

                        if (stalenessInDays >= params.staleThreshold) {
                            staleBranches.add("${branchName} - Stale for ${stalenessInDays} days")
                        }
                    }

                    echo "Stale Branches:"
                    echo "${staleBranches.join('\n')}"
                }
            }
        }

        stage('Delete Stale Branches') {
            steps {
                script {
                    def currentTimestamp = new Date().toInstant().toEpochMilli()

                    sh(script: 'git ls-remote --heads origin', returnStdout: true).eachLine { branchLine ->
                        def branchInfo = branchLine.split()
                        def branchName = branchInfo[1].replaceAll(/.*refs\/heads\//, '').trim()
                        def lastCommitTimestamp = branchInfo[0].toLong() * 1000

                        def stalenessInDays = (currentTimestamp - lastCommitTimestamp) / (24 * 60 * 60 * 1000)

                        if (stalenessInDays >= params.staleThreshold) {
                            echo "Deleting Stale Branch: ${branchName}"
                            sh "git push origin --delete ${branchName}"
                        }
                    }

                    if (staleBranches.isEmpty()) {
                        echo "No stale branches to delete."
                    }
                }
            }
        }
    }
}

//  Pipeline to update agol content
def notebook_scripts = ['R9_FiresNotebookDeploy.py']
// returns a list of changed files
@NonCPS
String getChangedFilesList() {

    changedFiles = []
    for (changeLogSet in currentBuild.changeSets) {
        for (entry in changeLogSet.getItems()) { // for each commit in the detected changes
            for (file in entry.getAffectedFiles()) {
                changedFiles.add(file.getPath()) // add changed file to list
            }
        }
    }
    return changedFiles

}

pipeline {
    agent { docker { image 'python:3.7.9' } }
    environment {
          agol_creds = credentials('agol_geoplatform')
    }
    stages {
        stage('dynamic stages') {
            steps {
                script {
                    notebook_scripts.each {nbscript ->
                        try {
                            stage("deploy ipynb $nbscript") {
                                // if (getChangedFilesList().contains(nbscript)){
                                    withEnv(["HOME=${env.WORKSPACE}"]) {
                                    sh 'python --version'
                                    sh 'pip install -r requirements.txt --user'
                                    sh "python -c 'from update_ipynb import clean_py_script; clean_py_script(\"$nbscript\")'"
                                    sh "python -m pytest tests/test_$nbscript"
                                    sh 'python -c "import sys; print(sys.path)"'
                                    sh("python update_ipynb.py $agol_creds_USR $agol_creds_PSW $nbscript")
                                    }
                                // }
                            }
                        } catch (Exception e) {
                            echo e.toString()

                        }
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        failure {
            echo 'failure'
        // alert
        }
        success {
            echo 'success'
        // alert
        }
    }
}


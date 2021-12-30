def notebook_scripts = ['R9_FiresNotebookDeploy.py', 'test']

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
        stage('checkout scm'){
            steps{
                checkout scm

                echo getChangedFilesList()
            }
        }

        stage('dynamic ipynb deploy') {
            steps {
                script {
                    notebook_scripts.each {nbscript ->
                        stage("deploy ipynb $nbscript") {
//                             when { changeset "**/${nbscript}"}
                            if (getChangedFilesList().contains(nbscript)){
                                withEnv(["HOME=${env.WORKSPACE}"]) {
                                sh 'python --version'
                                sh 'pip install -r requirements.txt --user'
                                sh 'python -c "import sys; print(sys.path)"'
                                sh('python update_ipynb.py $agol_creds_USR $agol_creds_PSW ${file}')
                                }
                            }

                        }
                    }

                }
            }
        }

//         stage('deploy_ipynb') {
//             when { changeset "**/${notebook_script}"}
//                 steps {
//                     script {
//                         notebooks = ["R9_FiresNotebookDeploy.py"]
//                         for (file in notebooks) {
//                             echo '${file}'
//                             if (changeset "**/${file}") {
//                                 withEnv(["HOME=${env.WORKSPACE}"]) {
//                                 sh 'python --version'
//                                 sh 'pip install -r requirements.txt --user'
//                                 sh 'python -c "import sys; print(sys.path)"'
//                                 // sh 'jupytext --to notebook R9_Fires.py'
//                                 sh('python update_ipynb.py $agol_creds_USR $agol_creds_PSW ${file}')
//
//                             }
//                         }
//
//                     }
//                 }
//             }
//         }
    }
    post {
        always {

            cleanWs()
        }
    }
}


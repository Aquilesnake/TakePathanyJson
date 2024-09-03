pipeline {
    agent any
    stages {
        stage('Scan and Extract Data') {
            steps {
                script {
                    def environments = ['cl-ist-ia4', 'cl-ist-ia9', 'cl-uat-pa5']
                    def csvContent = "Environment,Path,Project,AppName,Branch,Commit\n"

                    for (env in environments) {
                        def dirPath = "environment/${env}/manifests"
                        echo "Processing environment: ${env}"
                        
                        if (fileExists(dirPath)) {
                            echo "Searching for job-metadata.json files in: ${dirPath}"
                            
                            // Define the command to find .json files
                            def findCmd = """cmd /c "for /R ${dirPath} %i in (job-metadata.json) do @echo %i" """
                            
                            // Run the command and capture the output
                            def files = bat(script: findCmd, returnStdout: true).trim().split("\n")
                            
                            if (files.length > 0) {
                                echo "Found ${files.length} files"
                                for (file in files) {
                                    def jsonContent = readFile(file)
                                    def json = readJSON text: jsonContent
                                    csvContent += "${env},${file},${json.project},${json.appname},${json.branch},${json.commit}\n"
                                }
                            } else {
                                echo "No job-metadata.json files found in ${dirPath}"
                            }
                        } else {
                            echo "Warning: Directory ${dirPath} does not exist. Skipping this environment."
                        }
                    }

                    // Write the CSV content to a file
                    writeFile file: 'output.csv', text: csvContent
                    echo "CSV Content:\n${csvContent}"
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'output.csv', allowEmptyArchive: true
        }
    }
}

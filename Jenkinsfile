pipeline {
    agent any

    environment {
        REPO_NAME = 'Test_createlistfromPath'
        REPO_URL = "https://github.com/Aquilesnake/${REPO_NAME}.git"
        BASE_PATH = "environment/"
        ALLOWED_ENVIRONMENTS = "cl-ist-ia4,cl-ist-ia9,cl-uat-pa5"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    deleteDir()
                    git url: REPO_URL, branch: 'main'
                    bat "dir /B /S"
                    echo "Repository cloned successfully"
                }
            }
        }
        
        stage('Scan and Extract Data') {
            steps {
                script {
                    def csvContent = "Environment,Path,Project,AppName,Branch,Commit\n"
                    def environments = ALLOWED_ENVIRONMENTS.split(',')
                    def totalFilesProcessed = 0
                    
                    echo "Environments to process: ${environments}"
                    
                    environments.each { env ->
                        echo "Processing environment: ${env}"
                        def envPath = "${BASE_PATH}${env}/manifests"
                        echo "Searching for job-metadata.json files in: ${envPath}"
                        
                        // Check if the directory exists
                        if (!fileExists(envPath)) {
                            echo "Warning: Directory ${envPath} does not exist. Skipping this environment."
                            return // Skip to the next iteration of the loop
                        }
                        
                        // Find all job-metadata.json files recursively
                        def findCommand = "for /R \"${envPath}\" %i in (job-metadata.json) do @echo %i"
                        echo "Executing command: ${findCommand}"
                        def metadataFiles = bat(script: findCommand, returnStdout: true).trim()
                        
                        if (metadataFiles.isEmpty()) {
                            echo "No job-metadata.json files found in ${envPath}"
                            return // Skip to the next iteration of the loop
                        }
                        
                        echo "Found the following job-metadata.json files:"
                        echo metadataFiles
                        
                        metadataFiles.split('\n').each { filePath ->
                            echo "Processing file: ${filePath}"
                            def relativePath = filePath.replace(BASE_PATH, '')
                            def pathParts = relativePath.tokenize('\\')
                            def projectName = pathParts.size() > 2 ? pathParts[1] : ''
                            def appName = pathParts.size() > 3 ? pathParts[2] : ''
                            def parentPath = pathParts[0..-2].join('/')
                            
                            try {
                                def fileContent = readFile(file: filePath)
                                echo "Contents of ${filePath}:"
                                echo fileContent
                                
                                def metadataJson = readJSON text: fileContent
                                
                                def project = metadataJson.project ?: projectName
                                def app = metadataJson.appname ?: appName
                                def branch = metadataJson.branch ?: ''
                                def commit = metadataJson.commit ?: ''
                                
                                def csvLine = "${env},${parentPath},${project},${app},${branch},${commit}"
                                echo "Adding to CSV: ${csvLine}"
                                csvContent += "${csvLine}\n"
                                totalFilesProcessed++
                            } catch (Exception e) {
                                echo "Error processing file ${filePath}: ${e.message}"
                                echo "Stack trace: ${e.stackTrace.join('\n')}"
                            }
                        }
                    }
                    
                    echo "Total files processed: ${totalFilesProcessed}"
                    echo "Writing CSV content to file"
                    writeFile file: 'output.csv', text: csvContent
                    
                    echo "CSV Content:"
                    echo csvContent
                    
                    echo "Verifying CSV file content:"
                    bat "type output.csv || exit 0"
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'output.csv', fingerprint: true
            echo "Pipeline completed successfully. CSV file archived."
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
    }
}

pipeline {
    agent any

    environment {
        REPO_NAME = 'Test_createlistfromPath'
        REPO_URL = "https://github.com/Aquilesnake/${REPO_NAME}.git"
        BASE_PATH = "environment/"
        ALLOWED_ENVIRONMENTS = "var1-var2-var3"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    deleteDir()
                    git url: REPO_URL, branch: 'main'
                    sh "ls -la"
                    echo "Repository cloned successfully"
                }
            }
        }
        
        stage('Scan and Extract Data') {
            steps {
                script {
                    def csvContent = "Environment,Path,Project,AppName,Branch,Commit\n"
                    def environments = ALLOWED_ENVIRONMENTS.split(',')
                    
                    environments.each { env ->
                        echo "Processing environment: ${env}"
                        def envPath = "${BASE_PATH}${env}/manifests"
                        
                        // Check if the directory exists
                        if (!fileExists(envPath)) {
                            echo "Warning: Directory ${envPath} does not exist. Skipping this environment."
                            return // Skip to the next iteration of the loop
                        }
                        
                        // Find all job-metadata.json files recursively
                        def findCommand = "find ${envPath} -type f -name 'data.json'"
                        def findResult = sh(script: findCommand, returnStdout: true).trim()
                        
                        if (findResult) {
                            echo "Found job-metadata.json files:"
                            echo findResult
                            
                            findResult.split('\n').each { filePath ->
                                echo "Processing file: ${filePath}"
                                try {
                                    def fileContent = readFile(file: filePath)
                                    def jsonData = parseJSON(fileContent)
                                    
                                    def relativePath = filePath.minus(BASE_PATH)
                                    def pathParts = relativePath.tokenize('/')
                                    def project = jsonData.proyect ?: (pathParts.size() > 2 ? pathParts[2] : '')
                                    def appName = jsonData.appname ?: (pathParts.size() > 3 ? pathParts[3] : '')
                                    def branch = jsonData.branch ?: ''
                                    def commit = jsonData.commit ?: ''
                                    
                                    def csvLine = "${env},${relativePath},${project},${appName},${branch},${commit}"
                                    echo "Adding to CSV: ${csvLine}"
                                    csvContent += "${csvLine}\n"
                                } catch (Exception e) {
                                    echo "Error processing file ${filePath}: ${e.message}"
                                }
                            }
                        } else {
                            echo "No job-metadata.json files found in ${envPath}"
                        }
                    }
                    
                    echo "Writing CSV content to file"
                    writeFile file: 'output.csv', text: csvContent
                    
                    echo "CSV Content:"
                    echo csvContent
                    
                    echo "Verifying CSV file content:"
                    sh "cat output.csv || true"
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

def parseJSON(String json) {
    def slurper = new groovy.json.JsonSlurper()
    return slurper.parseText(json)
}

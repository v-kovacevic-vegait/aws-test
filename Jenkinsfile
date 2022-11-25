pipeline{
    agent any
    stages{
        stage('Stop old container'){
            steps{
                script {
                    def containerExists = sh script: "docker inspect -f '{{.State.Running}}' mealforfamily-test-mealforfamily-1", returnStatus: true
                    if( containerExists == 0){      
                        echo 'docker stop mealforfamily-test-mealforfamily-1'
                        echo 'Old container stopped'
                    }else{
                        echo 'mealforfamily-test-mealforfamily-1 NOT FOUND'
                    }
                }
            }
        }
        stage('Build new container'){
            steps{
                script {
					sh "pwd"
					sh "ls -a"
					echo 'docker-compose up --build -d mealforfamily'
					echo 'New container built'
                }
            }
        }
        stage('Update-database'){
            steps{
                script {
                    runDataUpdate()
                }
            }
        }
        stage('Delete old container'){
            steps{
                script {
                    def containerExists = sh script: "docker inspect -f '{{.State.Running}}' mealforfamily-test-mealforfamily-1-new", returnStatus: true
                    def oldContainerExists = sh script: "docker inspect -f '{{.State.Running}}' mealforfamily-test-mealforfamily-1", returnStatus: true
                    if( containerExists == 0){
                        echo 'mealforfamily-test-mealforfamily-1-new FOUND'
                        if( oldContainerExists == 0){
                            echo 'docker rm mealforfamily-test-mealforfamily-1'
                            echo 'Old container deleted'
                        }else{
                            echo 'mealforfamily-test-mealforfamily-1 NOT FOUND'
                        }
                    }else{
                        echo 'mealforfamily-test-mealforfamily-1-new NOT FOUND'
                    }
                }
            }
        }
        stage('Rename new container'){
            steps{
                script {
                    def containerExists = sh script: "docker inspect -f '{{.State.Running}}' mealforfamily-test-mealforfamily-1-new", returnStatus: true
                    if( containerExists == 0){
                        echo 'docker rename mealforfamily-test-mealforfamily-1-new mealforfamily-test-mealforfamily-1'
                        echo 'New container renamed to mealforfamily-test-mealforfamily-1'
                    }else{
                        echo 'mealforfamily-test-mealforfamily-1-new NOT FOUND'
                    }
                }
            }
        }
        stage('Docker system prune'){
            steps{
                script {
                    echo "docker system prune -f"
                    echo 'Docker system prune'
                }
            }
        }
    }
}

@NonCPS
def runDataUpdate() {
    
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
           for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            echo "${entry.commitId} by ${entry.author} on ${new Date(entry.timestamp)}: ${entry.msg}"
            def files = new ArrayList(entry.affectedFiles)
            for (int k = 0; k < files.size(); k++) {
                def file = files[k]
                echo "  ${file.editType.name} ${file.path}"
                if(file.path.contains("Migrations")){
                    sh '''
					pwd
                    #!/bin/bash
                    export PATH="$PATH:$HOME/.dotnet/tools/"
                    dotnet ef database update --connection "User ID =test;Password=test;Host=172.0.0.1;Port=5432;Database=mealforfamilydb";
                    '''
                    echo "Database updated"
                }
           }
        }
    }
}

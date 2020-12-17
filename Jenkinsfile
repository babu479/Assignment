///import groovy.json.JsonSlurper
def runShell(String command){
    def responseCode = sh returnStatus: true, script: "${command} &> tmp.txt"
    def output =  readFile(file: "tmp.txt")
    return (output != "")
}
pipeline {
    agent {
        label 'master'
    }
    parameters {
        booleanParam( name: 'Static_Check',description: 'Static_Check',defaultValue: false,)
        booleanParam(name: 'QA', description: 'QA',defaultValue: false )
        booleanParam(name: 'Unit_Test', description: 'Unit_Test',defaultValue: false )
        string(name: 'Success_Email',defaultValue: 'manasjit.mohanty@yahoo.com', description: 'Replace the email as required')
        string(name: 'Failure_Email', defaultValue: 'manasjit.mohanty@yahoo.com', description: 'Replace the email as required')
        //string(defaultValue: 'manasjit.mohanty@gmail.com', description: 'Success_Email', name: 'Success_Email')
        //string(defaultValue: 'manasjit.mohanty@gmail.com', description: 'Failure_Email', name: 'Failure_Email')
    }
    options {
        timestamps()
    }
    tools {
        maven 'M3'
        git 'git'
    }
    stages {
        stage ('Git pull'){
            steps{
                checkout scm
                echo '${STAGE_NAME}' >> $WORKSPACE/stageExecuted
            }
        }
        stage ('Is this required?'){
            steps {
                echo "http rest to be executed"
                script {
                    date = new Date()
                    todays_date = date.format("yyyy-MM-dd", TimeZone.getTimeZone('UTC'))
                    year = date.format("yyyy", TimeZone.getTimeZone('UTC'))
                    println year
                    
                    def response = httpRequest "https://calendarific.com/api/v2/holidays?&api_key=758f54db8c52c2b500c928282fe83af1b1aa2be8&country=IN&year=$year"
                    println("Status: "+response.status)
                    //println("Content: "+response.content)
                    node {
                        writeFile file: 'object.json', text: response.content
                        sh 'cat object.json'


                    if (runShell('grep \'$todays_date\' object.json')) {
                    //if (response.content.hasVariable($todays_date)) {
                        holiday = true
                    }
                    else {
                        holiday = false
                    }
                    }
                    
                }
                echo '${STAGE_NAME}' >> $WORKSPACE/stageExecuted
                
            }
        }
        stage ('Build'){
            when {
                allOf {
                    expression { holiday == false }
                }
            }
            steps {
                echo "Build step"
                script {
                    writeFile file: 'Build.txt', text: 'This stage will do the build and create the artifacts'
                    sh 'cat Build.txt'
                    writeFile file: 'Static_Check.txt', text: 'This stage will do the check foe the code qulaity whether it followd the coding standard or not'
                    sh 'cat Static_Check.txt'
                    writeFile file: 'QA.txt', text: 'This stage will do the Quality analysis for the code'
                    sh 'cat QA.txt'
                    writeFile file: 'Unit_Test.txt', text: 'This stage will do the Unit test for the code whter the basic functionality is working or not'
                    sh 'cat Unit_Test.txt'
                    sh 'pwd'
                    sh 'ls -lrt'
                    echo '${STAGE_NAME}' >> $WORKSPACE/stageExecuted
                }
            }
        }
        stage ('QualityCheck and Unit Test'){
            parallel {
                stage ('QualityCheck'){
                    agent {
                        label 'master'
                    }
                    stages {
                        stage ('Static_Check'){
                            when {
                                allOf {
                                    expression { params.Static_Check == true && holiday == false }
                                }
                            }
                            steps{
                                echo "Static Check"
                                echo '${STAGE_NAME}' >> $WORKSPACE/stageExecuted
                            }
                        }
                        stage ('QA'){
                            when {
                                allOf {
                                    expression { params.QA == true && holiday == false }
                                }
                            }
                            steps {
                                echo "QA Done"
                                echo '${STAGE_NAME}' >> $WORKSPACE/stageExecuted
                            }
                        }
                    }

                }
                stage ('UnitTest'){
                    when {
                        allOf {
                            expression { params.Unit_Test == true && holiday == false}
                        }
                    }
                    agent {
                        label 'master'
                    }
                    steps {
                        echo "UT done"
                        echo '${STAGE_NAME}' >> $WORKSPACE/stageExecuted
                    }
                }
            }
        }
        stage ('Summary'){
            steps {
                echo "Print summary of all stages"
                sh """ cat $WORKSPACE/stageExecuted"""
            }
        }

    }
    post {
        always {
            echo "Build completed"
        }
        failure {
            echo "Build failed"
            mail to: params.Failure_Email,
                subject: "Failed condition: ${currentBuild.fullDisplayName}",
                body: "Failed ${env.BUILD_URL}"
        }
        success {
            echo "Build passed"
            mail to: params.Success_Email,
                subject: "Success condition: ${currentBuild.fullDisplayName}",
                body: "Success ${env.BUILD_URL}"
        }
    }
}

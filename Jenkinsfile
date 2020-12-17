import groovy.json.JsonSlurper
def holiday
pipeline {
    agent {
        label 'master'
    }
    parameters {
        booleanParam( name: 'Static_Check',description: 'Static_Check',defaultValue: false,)
        booleanParam(name: 'QA', description: 'QA',defaultValue: false )
        booleanParam(name: 'Unit_Test', description: 'Unit_Test',defaultValue: false )
        string(defaultValue: '', description: 'Success_Email', name: 'Success_Email')
        string(defaultValue: '', description: 'Failure_Email', name: 'Failure_Email')
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
            }
        }
        stage ('Is this required?'){
            steps {
                echo "http rest to be executed"
                script {
                    date = new Date()
                    current_date = date.format("yyyy-MM-dd", TimeZone.getTimeZone('UTC'))
                    year = date.format("yyyy", TimeZone.getTimeZone('UTC'))
                    def response = httpRequest "https://calendarific.com/api/v2/holidays?&api_key=758f54db8c52c2b500c928282fe83af1b1aa2be8&country=IN&year=$year"
                    def json = new JsonSlurper().parseText(response.content)
                    //println("Status: "+response.status)
                    //println("Content: "+response.content)
                    node {
                        writeFile file: 'object.json', text: response.content
                        sh 'cat object.json'
                        if (runShell('grep \'$todays_date\' object.json')) {
                            holiday = true
                        }
                        else {
                            holiday = false
                        }
                    }
                }
                
            }
        }
        stage ('Build'){
            //when {
            //    allOf {
            //        expression { holiday == false }
            //    }
            //}
            steps {
                echo "Build step"
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
                                    expression { params.Static_Check == true && holiday == true}
                                }
                            }
                            steps{
                                echo "Static Check"
                            }
                        }
                        stage ('QA'){
                            when {
                                allOf {
                                    expression { params.QA == true && holiday == true}
                                }
                            }
                            steps {
                                echo "QA Done"
                            }
                        }
                    }

                }
                stage ('UnitTest'){
                    when {
                        allOf {
                            expression { params.Unit_Test == true && holiday == true}
                        }
                    }
                    agent {
                        label 'master'
                    }
                    steps {
                        echo "UT done"
                    }
                }
            }
        }
        stage ('Summary'){
            steps {
                echo "Print summary of all stages"
            }
        }

    }
    post {
        always {
            echo "Build completed"
        }
        failure {
            echo "Build failed"
        }
        success {
            echo "Build passed"
        }
    }
}

def runShell(String command){
    def responseCode = sh returnStatus: true, script: "${command} &> tmp.txt"
    def output =  readFile(file: "tmp.txt")
    return (output != "")
}

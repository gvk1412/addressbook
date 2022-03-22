pipeline{
    agent none
    tools{
        jdk 'myjava'
        maven 'mymaven'
    }
    parameters{
        choice(name:'VERSION',choices:['1.1.0','1.2.0','1.3.0'],description:'version of the code')
        booleanParam(name: 'executeTests',defaultValue: true,description:'tc validity')
    }
    stages{
        stage("COMPILE"){
            agent {label 'linux_slave'}
            steps{
                script{
                    echo "Compiling the code"
                    sh 'mvn compile'
                }
            }
        }
        stage("UNITTEST"){
            agent any
            when{
                expression{
                    params.executeTests == true
                }
            }
            steps{
                script{
                    echo "Testing the code"
                    sh 'mvn test'
                }
            }
            post{
                always{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
         stage("PACKAGE"){
             agent {label 'linux_slave'}
            when{
                expression{
                    BRANCH_NAME == 'master'
                }
            }
            steps{
                script{
                    echo "Packaging the code"
                    sh 'mvn package'
                }
            }
        }
         stage("BUILD THE DOCKER IMAGE"){
            agent any
             when{
                expression{
                    BRANCH_NAME == 'master'
                }
            }
            steps{
                script{
                    echo "BUILDING THE DOCKER IMAGE"
                    echo "Deploying version ${params.VERSION}"
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'sudo docker build -t devopstrainer/java-mvn-privaterepo:$BUILD_NUMBER .'
                        sh 'sudo sudo docker login -u $USER -p $PASS'
                        sh 'sudo docker push devopstrainer/java-mvn-privaterepo:$BUILD_NUMBER'
                }
            }
        }
         }
        stage("DEPLOY"){
            agent any
             when{
                expression{
                    BRANCH_NAME == 'master'
                }
            }
            steps{
                script{
                    echo "Deploying the app"
                    echo "Deploying version ${params.VERSION}"
                }
            }
    }
}
    }

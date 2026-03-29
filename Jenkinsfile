pipeline {
    agent any

    environment {
        CONTAINER_ID = ''
        SUM_PY_PATH = '/Users/pau-01/projet-docker/sum.py'
        DIR_PATH = '/Users/pau-01/projet-docker'
        TEST_FILE_PATH = '/Users/pau-01/projet-docker/test_variables.txt'
    }

    stages {

        stage('Build') {
            steps {
                script {
                    sh "docker build -t sum-app ${DIR_PATH}"
                }
            }
        }

        stage('Run') {
            steps {
                script {
                    def output = sh(script: 'docker run -d sum-app', returnStdout: true)
                    CONTAINER_ID = output.trim()
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def testLines = readFile(TEST_FILE_PATH).split('\n')
                    for (line in testLines) {
                        if (line.trim() == '') continue
                        def vars = line.trim().split(' ')
                        def arg1 = vars[0]
                        def arg2 = vars[1]
                        def expectedSum = vars[2].toFloat()

                        def output = sh(script: "docker exec ${CONTAINER_ID} python /app/sum.py ${arg1} ${arg2}", returnStdout: true)
                        def result = output.trim().toFloat()

                        if (result == expectedSum) {
                            echo "✅ Test réussi : ${arg1} + ${arg2} = ${result}"
                        } else {
                            error "❌ Test échoué : ${arg1} + ${arg2} = ${result}, attendu ${expectedSum}"
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh 'docker login -u donbeni -p Sagace1212@!'
                    sh 'docker tag sum-app donbeni/sum-app:latest'
                    sh 'docker push donbeni/sum-app:latest'
                }
            }
        }
    }

    post {
        always {
            script {
                sh "docker stop ${CONTAINER_ID}"
                sh "docker rm ${CONTAINER_ID}"
            }
        }
    }
}
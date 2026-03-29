pipeline {
    agent any

    environment {
        DIR_PATH = '/Users/pau-01/projet-docker'
        TEST_FILE_PATH = '/Users/pau-01/projet-docker/test_variables.txt'
    }

    stages {

        stage('Build') {
            steps {
                script {
                    sh "/usr/local/bin/docker build -t sum-app ${DIR_PATH}"
                }
            }
        }

        stage('Run') {
            steps {
                script {
                    def output = sh(script: '/usr/local/bin/docker run -d sum-app', returnStdout: true).trim()
                    echo "CONTAINER_ID: ${output}"
                    writeFile file: '/tmp/container_id.txt', text: output
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def containerId = readFile('/tmp/container_id.txt').trim()
                    def testLines = readFile(TEST_FILE_PATH).split('\n')
                    for (line in testLines) {
                        if (line.trim() == '') continue
                        def vars = line.trim().split(' ')
                        def arg1 = vars[0]
                        def arg2 = vars[1]
                        def expectedSum = vars[2].toFloat()
                        def output = sh(script: "/usr/local/bin/docker exec ${containerId} python /app/sum.py ${arg1} ${arg2}", returnStdout: true)
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
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '/usr/local/bin/docker login -u $DOCKER_USER -p $DOCKER_PASS'
                        sh '/usr/local/bin/docker tag sum-app donbeni/sum-app:latest'
                        sh '/usr/local/bin/docker push donbeni/sum-app:latest'
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def containerId = readFile('/tmp/container_id.txt').trim()
                sh "/usr/local/bin/docker stop ${containerId} || true"
                sh "/usr/local/bin/docker rm ${containerId} || true"
            }
        }
    }
}

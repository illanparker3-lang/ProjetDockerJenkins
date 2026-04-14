pipeline {
    agent any

    environment {
        IMAGE_NAME = "sum-image"
        CONTAINER_NAME = "sum-container"
        DIR_PATH = "."
        TEST_FILE_PATH = "test_variables.txt"
        DOCKERHUB_REPO = "illanparker3/sum-image"
    }

    stages {
        stage('Build') {
            steps {
                bat "docker build -t %IMAGE_NAME% %DIR_PATH%"
            }
        }

        stage('Run') {
            steps {
                bat "docker rm -f %CONTAINER_NAME% || exit 0"
                bat "docker run -d --name %CONTAINER_NAME% %IMAGE_NAME%"
                bat "docker ps -a"
            }
        }

        stage('Test') {
            steps {
                script {
                    def testLines = readFile(env.TEST_FILE_PATH).trim().split('\n')

                    for (line in testLines) {
                        def vars = line.trim().split(/\s+/)
                        def arg1 = vars[0]
                        def arg2 = vars[1]
                        def expectedSum = vars[2].toFloat()

                        def output = bat(
                            script: "@docker exec %CONTAINER_NAME% python /app/sum.py ${arg1} ${arg2}",
                            returnStdout: true
                        ).trim()

                        def result = output.readLines()[-1].trim().toFloat()

                        if (result == expectedSum) {
                            echo "Test réussi : ${arg1} + ${arg2} = ${result}"
                        } else {
                            error "Test échoué : ${arg1} + ${arg2}, attendu ${expectedSum}, obtenu ${result}"
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                bat "docker tag %IMAGE_NAME% %DOCKERHUB_REPO%"
                bat "docker push %DOCKERHUB_REPO%"
            }
        }
    }

    post {
        always {
            bat "docker stop %CONTAINER_NAME% || exit 0"
            bat "docker rm %CONTAINER_NAME% || exit 0"
        }
    }
}
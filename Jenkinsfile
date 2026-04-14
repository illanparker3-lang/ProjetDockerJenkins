pipeline {
    agent any

    environment {
        IMAGE_NAME = "sum-image"
        CONTAINER_ID = ""
        DIR_PATH = "."
        TEST_FILE_PATH = "test_variables.txt"
        DOCKERHUB_REPO = "ton_username_dockerhub/sum-image"
    }

    stages {
        stage('Build') {
            steps {
                bat "docker build -t %IMAGE_NAME% %DIR_PATH%"
            }
        }

        stage('Run') {
            steps {
                script {
                    def output = bat(
                        script: 'docker run -d %IMAGE_NAME%',
                        returnStdout: true
                    ).trim()

                    def lines = output.split('\n')
                    env.CONTAINER_ID = lines[-1].trim()

                    echo "Conteneur lancé : ${env.CONTAINER_ID}"
                }
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
                            script: "docker exec ${env.CONTAINER_ID} python /app/sum.py ${arg1} ${arg2}",
                            returnStdout: true
                        ).trim()

                        def resultLines = output.split('\n')
                        def result = resultLines[-1].trim().toFloat()

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
            script {
                if (env.CONTAINER_ID?.trim()) {
                    bat "docker stop ${env.CONTAINER_ID}"
                    bat "docker rm ${env.CONTAINER_ID}"
                }
            }
        }
    }
}
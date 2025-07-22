pipeline {
    agent any

    stages {
        stage('Clone repository') {
            steps {
                cleanWs()
                git url: 'https://github.com/EndlessEmptiness1nside/juice-shop.git', branch: 'master'
            }
        }

        stage('Check and Install Dependencies') {
            steps {
                script {
                    docker.image('node:20').inside {
                        // Задаем переменные окружения для Cypress и npm, чтобы они создавали кэш локально
                        withEnv(["CYPRESS_CACHE_FOLDER=./.cache/Cypress"]) {
                            if (fileExists('node_modules')) {
                                echo "Зависимости уже установлены, пропуск установки."
                            } else {
                                echo "Папка node_modules не найдена. Установка зависимостей..."
                                sh 'npm install --cache ./.npm-cache --no-update-notifier'
                            }
                        }
                    }
                }
            }
        }

        stage('Semgrep Scan') {
            steps {
                script {
                    docker.image('semgrep/semgrep').inside {
                        echo "Запуск сканирования Semgrep..."
                        // Устанавливаем переменную SEMGREP_HOME, чтобы избежать проблем с правами доступа
                        withEnv(["SEMGREP_HOME=./.semgrep-home"]) {
                            sh 'semgrep scan --config="p/default" --error --output="semgrep-report.json" --json .'
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Архивация отчета Semgrep..."
            archiveArtifacts artifacts: 'semgrep-report.json', fingerprint: true
            
            echo "Пайплайн завершил работу."
        }
    }
}

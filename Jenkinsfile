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
                    // Устанавливаем переменную окружения для Cypress напрямую
                    docker.image('node:20').inside('-e CYPRESS_CACHE_FOLDER=./.cache/Cypress') {
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

        stage('Semgrep Scan') {
            steps {
                script {
                    // **ФИНАЛЬНОЕ РЕШЕНИЕ:**
                    // Устанавливаем переменную HOME равной текущей рабочей директории.
                    // Это заставит Semgrep создать свою папку .semgrep внутри нашего проекта.
                    docker.image('semgrep/semgrep').inside('-e HOME=.') {
                        echo "Запуск сканирования Semgrep..."
                        sh 'semgrep scan --config="p/default" --error --output="semgrep-report.json" --json .'
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Архивация отчета Semgrep..."
            // allowEmptyArchive: true предотвращает ошибку, если отчет не был создан
            archiveArtifacts artifacts: 'semgrep-report.json', fingerprint: true, allowEmptyArchive: true
            
            echo "Пайплайн завершил работу."
        }
    }
}

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
                    // Используем .inside() с аргументом -e для установки переменных окружения
                    // для npm и Cypress
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
                    // Передаем переменную окружения SEMGREP_HOME напрямую в контейнер
                    // с помощью аргумента -e.
                    docker.image('semgrep/semgrep').inside('-e SEMGREP_HOME=./.semgrep-home') {
                        echo "Запуск сканирования Semgrep..."
                        sh 'semgrep scan --config="p/default" --error --output="semgrep-report.json" --json .'
                    }
                }
            }
        }
    }

    post {
        // Блок always гарантирует, что артефакты будут заархивированы
        // даже если этап сканирования завершится с ошибкой (например, из-за --error)
        always {
            echo "Архивация отчета Semgrep..."
            archiveArtifacts artifacts: 'semgrep-report.json', fingerprint: true, allowEmptyArchive: true
            
            echo "Пайплайн завершил работу."
        }
    }
}

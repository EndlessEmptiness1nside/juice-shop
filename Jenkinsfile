pipeline {
    agent any

    stages {
        stage('Clone repository') {
            steps {
                // Очищаем рабочее пространство перед клонированием
                cleanWs()
                // Клонируем репозиторий
                git url: 'https://github.com/EndlessEmptiness1nside/juice-shop.git', branch: 'master'
            }
        }

        stage('Check and Install Dependencies') {
            steps {
                script {
                    docker.image('node:20').inside {
                        if (fileExists('node_modules')) {
                            echo "Зависимости уже установлены, пропуск установки."
                        } else {
                            echo "Папка node_modules не найдена. Установка зависимостей..."
                            // Указываем npm использовать локальную папку для кэша, чтобы избежать проблем с правами доступа
                            sh 'npm install --cache ./.npm-cache --no-update-notifier'
                        }
                    }
                }
            }
        }

        stage('Semgrep Scan') {
            steps {
                script {
                    // Используем официальный образ Semgrep для сканирования
                    docker.image('semgrep/semgrep').inside {
                        echo "Запуск сканирования Semgrep..."
                        // Команда для сканирования с набором правил по умолчанию ('p/default')
                        // --error - завершить с ошибкой, если найдены уязвимости
                        // --json - выводить результат в формате JSON
                        // --output "semgrep-report.json" - сохранить отчет в файл
                        sh 'semgrep scan --config="p/default" --error --output="semgrep-report.json" --json .'
                    }
                }
            }
        }
    }

    post {
        always {
            // Этот блок выполняется всегда в конце сборки
            echo "Архивация отчета Semgrep..."
            // Архивируем созданный отчет для доступа из интерфейса Jenkins
            archiveArtifacts artifacts: 'semgrep-report.json', fingerprint: true
            
            echo "Пайплайн завершил работу."
        }
    }
}

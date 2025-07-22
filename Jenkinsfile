pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/juice-shop/juice-shop.git'
            }
        }

        stage('Проверка и установка зависимостей') {
            steps {
                script {
                    // Проверяем, существует ли директория node_modules
                    if (!fileExists('node_modules')) {
                        echo 'Зависимости не найдены. Запускается npm install...'
                        // Для установки зависимостей используется образ Node.js
                        docker.image('node:20').inside {
                            sh 'npm install'
                        }
                    } else {
                        echo 'Зависимости уже установлены.'
                    }
                }
            }
        }

        stage('Сканирование Semgrep') {
            steps {
                script {
                    // Используем официальный образ Semgrep для сканирования
                    // Ключ --json указывает на формирование отчета в формате JSON
                    docker.image('semgrep/semgrep').inside {
                        sh 'semgrep --config="p/default" --error --output="semgrep-report.json" --json .'
                    }
                }
            }
        }

        stage('Архивация отчета') {
            steps {
                archiveArtifacts artifacts: 'semgrep-report.json', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Пайплайн завершил работу.'
        }
    }
}

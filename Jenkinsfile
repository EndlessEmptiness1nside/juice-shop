pipeline {
    agent any
    environment {
        // Путь к Node.js 22
        PATH = "/usr/local/nodejs-22/bin:${env.PATH}"
        NVM_DIR = "$HOME/.nvm"
        PROJECT_DIR = "/var/lib/jenkins/workspace/JUICE_SHOP"
    }
    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm
            }
        }
        stage('Install Node.js') {
            steps {
                script {
                    // Установка NVM и Node.js 22 в одной команде
                    sh """
                    # Установка NVM
                    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh  | bash
                    
                    # Загрузка NVM
                    export NVM_DIR="$HOME/.nvm"
                    [ -s "\$NVM_DIR/nvm.sh" ] && \\. "\$NVM_DIR/nvm.sh"
                    
                    # Установка и использование Node.js 22
                    nvm install 22
                    nvm use 22
                    node -v
                    npm -v
                    """
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    // Проверка наличия node_modules
                    def dependenciesInstalled = sh(
                        script: 'test -d node_modules',
                        returnStatus: true
                    ) == 0
                    
                    if (!dependenciesInstalled) {
                        echo 'Зависимости не установлены. Начинаю установку...'
                        
                        // Установка зависимостей с флагами
                        sh 'npm install --no-fund --no-audit --legacy-peer-deps'
                        
                        // Обновление уязвимых пакетов
                        sh 'npm install jsonwebtoken@latest multer@latest eslint@8.57.1'
                        
                        // Исправление уязвимостей
                        sh 'npm audit fix --legacy-peer-deps'
                    } else {
                        echo 'Зависимости уже установлены. Пропускаю установку...'
                    }
                }
            }
        }
        stage('Run Semgrep Scan') {
            steps {
                script {
                    // Установка Semgrep (если не установлен)
                    def semgrepInstalled = sh(
                        script: 'command -v semgrep',
                        returnStatus: true
                    ) == 0
                    
                    if (!semgrepInstalled) {
                        echo 'Semgrep не установлен. Начинаю установку...'
                        sh 'npm install -g @semgrep/cli'
                    }
                    
                    // Запуск сканирования и сохранение отчета в JSON
                    sh """
                    semgrep --config=auto \\
                            --output=${env.PROJECT_DIR}/semgrep_report.json \\
                            --json \\
                            ${env.PROJECT_DIR}
                    """
                    
                    // Проверка наличия отчета
                    sh 'ls -l ${env.PROJECT_DIR}/semgrep_report.json'
                }
            }
        }
        stage('Archive Report') {
            steps {
                // Сохранение JSON-отчета как артефакта
                archiveArtifacts artifacts: 'semgrep_report.json', allowEmptyArchive: false
            }
        }
    }
    post {
        success {
            echo 'Сканирование завершено успешно. Отчет сохранен в semgrep_report.json'
        }
        failure {
            echo 'Ошибка сканирования. Проверьте логи.'
        }
    }
}

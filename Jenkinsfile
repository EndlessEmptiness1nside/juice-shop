pipeline {
    agent any
    environment {
        // Укажите версию Node.js 20+ (путь зависит от вашей системы)
        PATH = "/usr/local/nodejs-22/bin:${env.PATH}"
        
        // Настройки прокси (если требуется; удалите, если не используется)
        NPM_CONFIG_PROXY = "http://<proxy-url>:<port>"
        NPM_CONFIG_HTTPS_PROXY = "http://<proxy-url>:<port>"
        
        // Увеличенный таймаут для npm
        NPM_CONFIG_TIMEOUT = "300000"
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
                    // Установка Node.js 22 через nvm (если не установлен)
                    sh 'curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh  | bash'
                    sh 'touch $HOME/.bashrc'
                    sh 'source ~/.bashrc && nvm install 22 && nvm use 22'
                    sh 'node -v && npm -v'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    // Очистка кэша npm (на случай ошибок)
                    sh 'npm cache clean --force'
                    
                    // Установка зависимостей с флагами для уменьшения предупреждений
                    sh 'npm install --no-fund --no-audit'
                    
                    // Обновление уязвимых пакетов
                    sh 'npm install jsonwebtoken@latest multer@latest eslint@latest'
                    
                    // Исправление уязвимостей через npm audit
                    sh 'npm audit fix'
                }
            }
        }
        stage('Build & Run Application') {
            steps {
                script {
                    // Сборка (если требуется)
                    sh 'npm run build'
                    
                    // Запуск приложения
                    sh 'npm start'
                }
            }
        }
    }
    post {
        success {
            echo 'Пайплайн успешно завершен!'
        }
        failure {
            echo 'Ошибка в пайплайне. Проверьте логи.'
        }
    }
}

pipeline {
    agent any
    environment {
        // Укажите путь к Node.js 22+ (если используется системный Node.js)
        PATH = "/usr/local/nodejs-22/bin:${env.PATH}"
        
        // Путь к NVM
        NVM_DIR = "$HOME/.nvm"
        
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
                    // Создайте .bashrc, если он отсутствует
                    sh 'touch $HOME/.bashrc'
                    
                    // Установка NVM без автоматического добавления в профиль
                    sh 'curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh  | bash || true'
                    
                    // Вручную добавьте настройки NVM в .bashrc
                    sh 'echo "export NVM_DIR=\\"$HOME/.nvm\\"" >> $HOME/.bashrc'
                    sh 'echo "[ -s \\"$NVM_DIR/nvm.sh\\"" && . "$NVM_DIR/nvm.sh" # Loads nvm" >> $HOME/.bashrc'
                    
                    // Перезагрузите окружение с использованием `.` вместо `source`
                    sh '. "$HOME/.bashrc"'
                    
                    // Установите Node.js 22 через NVM
                    sh '. "$NVM_DIR/nvm.sh" && nvm install 22 && nvm use 22'
                    sh 'node -v && npm -v'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    // Очистка кэша npm
                    sh 'npm cache clean --force'
                    
                    // Установка зависимостей с флагами для уменьшения предупреждений
                    sh 'npm install --no-fund --no-audit'
                    
                    // Обновление уязвимых пакетов
                    sh 'npm install jsonwebtoken@latest multer@latest eslint@latest'
                    
                    // Исправление уязвимостей
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

node {

    // --- Tool Initialization (Scripted syntax) ---
    def nodeHome = tool name: 'node16', type: 'nodejs'
    env.PATH = "${env.PATH}:${nodeHome}/bin:/opt/sonar-scanner/bin"

    try {

        stage('Clone Repository') {
            echo "Cloning React frontend repo..."
            git branch: 'main', url: 'https://github.com/OT-MICROSERVICES/frontend'
        }

        stage('Install Dependencies') {
            echo "Installing Node dependencies..."
            sh 'npm install'
        }

        stage('ESLint Static Code Analysis') {
            echo "Running ESLint analysis..."
            sh 'npx eslint src || true'
        }

        stage('SonarQube Analysis') {
            echo "Running SonarQube scanner..."

            withSonarQubeEnv('sonarqube-server') {
                sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=frontend \
                    -Dsonar.projectName=frontend \
                    -Dsonar.sources=. \
                    -Dsonar.exclusions=node_modules/** \
                    -Dsonar.host.url=$SONAR_HOST_URL \
                    -Dsonar.login=$SONAR_AUTH_TOKEN
                '''
            }
        }

        stage("Quality Gate") {
            timeout(time: 5, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: false
            }
        }

        echo "React static analysis pipeline SUCCESS!"

    } catch (err) {

        echo "Pipeline FAILED. Please check ESLint or SonarQube results."
        throw err

    } finally {
        echo "Pipeline execution completed!"
    }
}

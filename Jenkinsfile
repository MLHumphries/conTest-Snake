pipeline 
{
    agent none
    stages 
    {
        stage('CLONE GIT REPOSITORY') 
        {
            agent 
            {
                label 'ubuntu-appServer-conTest'
            }
            steps 
            {
                checkout scm
            }
        }  
 
        stage('SCA-SAST-SNYK-TEST') 
        {
            agent any
            steps 
            {
                script 
                {
                    snykSecurity(snykInstallation: 'Snyk', snykTokenId: 'snyk_credentials', severity: 'critical')
                }
            }
        }
 
        stage('SonarQube Analysis') 
        {
            agent 
            {
                label 'ubuntu-appServer-conTest'
            }
            steps 
            {
                script 
                {
                    def scannerHome = tool 'sonarqube'
                    withSonarQubeEnv('sonarqube') 
                    {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=gameapp \
                            -Dsonar.sources=."
                    }
                }
            }
        }
 
        stage('BUILD-AND-TAG') 
        {
            agent 
            {
                label 'ubuntu-appServer-conTest'
            }
            steps 
            {
                script 
                {
                    def app = docker.build("mlhumphries/contest-snake")
                    app.tag("latest")
                }
            }
        }
 
        stage('POST-TO-DOCKERHUB') 
        {    
            agent 
            {
                label 'ubuntu-appServer-conTest'
            }
            steps {
                script 
                {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_credentials') {
                        def app = docker.image("mlhumphries/contest-snake")
                        app.push("latest")
                    }
                }
            }
        }
 
        stage('DEPLOYMENT') 
        {    
            agent 
            {
                label 'ubuntu-appServer-conTest'
            }
            steps 
            {
                sh "docker-compose down"
                sh "docker-compose up -d"   
            }
        }
    }
}
pipeline { 
    agent any  
    stages { 
        stage('Build') { 
            steps { 
               echo 'This is a Build stage From a minimal pipeline.' 
            }
        }
        stage('Unit Tests') { 
            steps { 
               echo 'This is a Unit Tests run stage.' 
            }
        }
        stage('Sonar') { 
            steps { 
               echo 'This is a sonar check stage.' 
            }
        }
        stage('Gate Way') { 
            steps { 
               echo 'This is a gateway check stage.' 
            }
        }
    }
}
pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                // Obtener código del repo
                git 'https://github.com/miguelferrercrespo/helloworld.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Eyyy, esto es Python. No hay que compilar nada!!!'
                echo "WORKSPACE: ${env.WORKSPACE}"
                bat 'dir'
            }
        }

       stage('Tests') {
            parallel {
                stage('Unit') {
                    steps {
                        bat '''
                            set PYTHONPATH=%WORKSPACE%
                             pytest --junitxml=result-unit.xml test\\unit
                            '''
                         }
                    }

                     stage('Service') {
                        steps {
                            bat '''
                                set FLASK_APP=app\\api.py
                                set FLASK_ENV=development
                                start flask run
                                start java -jar C:\\Users\\migue\\Unir_Devops\\wiremock\\wiremock-jre8-standalone-2.28.0.jar --port 9090 --root-dir test\\wiremock
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test\\rest
                            '''
                        }
                    }
                }
            }
		
		stage('Cobertura') {
                steps {
                    bat '''
                        set PYTHONPATH=%WORKSPACE%
                        coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit
                        coverage xml
                     '''
                     recordCoverage qualityGates: [[criticality: 'NOTE', integerThreshold: 85, metric: 'LINE', threshold: 85.0], [criticality: 'ERROR', integerThreshold: 60, metric: 'LINE', threshold: 60.0], [criticality: 'NOTE', integerThreshold: 85, metric: 'BRANCH', threshold: 85.0], [criticality: 'ERROR', integerThreshold: 60, metric: 'BRANCH', threshold: 60.0]], tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']]
                }
            }

        stage('Results') {
            steps {
                junit 'result*.xml'
            }
        }
    }
}

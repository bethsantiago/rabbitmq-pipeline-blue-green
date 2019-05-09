pipeline {
    agent{
        kubernetes{
            label 'checklist-core'
            yamlFile './checklist-core/kubernetes/yaml/pod.yaml'
        }
    }

    environment {
        WORK_DIR = './checklist-core/'
        SUREFIRE_REGEX = '**/target/surefire-reports/TEST-*.xml'
        FAILSAFE_REGEX = '**/target/failsafe-reports/TEST-*.xml'
      }

    stages {

        stage('Prepare') {
          steps {
            git branch: env.BRANCH_NAME,
              credentialsId: 'suitelog',
              url: 'http://tfs2015.totvs.com.br:8080/tfs/CD-JV/_git/SL-MeuChecklist'
            }
        }

        stage('Compile') {
            steps {
                container('maven') {
                    sh 'mvn clean package -N -f ' + env.WORK_DIR + 'pom.xml' + ' -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                parallel(
                    'Unit': {
                        container('maven') {
                            sh 'mvn surefire:test -f ' + env.WORK_DIR + 'pom.xml'
                            junit SUREFIRE_REGEX
                        }
                    },
                    'Integration': {
                        echo "This is branch b"
                            //container('maven') {
                            //    sh 'mvn failsafe:integration-test -f ' + env.WORK_DIR + 'pom.xml'
                            //    junit FAILSAFE_REGEX
                            //}
                      }
                )
            }
        }

        stage('SonarQube analysis') {
            steps {
                container('sonar-scanner') {

                    script {
                        sh '''
                           sonar-scanner \
                            -Dsonar.projectKey=checklist-core \
                            -Dsonar.projectName="Checklist :: Core" \
                            -Dsonar.login=24c691e1190d40474698c991c2428f7c8bba9404 \
                            -Dsonar.projectBaseDir=./checklist-core \
                            -Dsonar.sources=./src/main/java \
                            -Djava-module.sonar.projectName="Checklist :: Core" \
                            -Djava-module.sonar.sources=src/main/java \
                            -Dsonar.language=java \
                            -Dsonar.java.binaries=target/ \
                            -Dsonar.host.url=http://sonar.sonar.svc.cluster.local:9000
                        '''
                      }
                }
            }
        }

        stage('Docker build') {
        
            when{
              branch 'master'
            }

            steps {
                container('docker') {

                    script {
                        def pom = getInfoPOM()
                        def imageName = getImageName('imageName')
                        def buildArgs = 'JAR_FILE=' + env.WORK_DIR + 'target/' + pom.artifactId + '-' + pom.version + '.jar'
                        sh 'docker build -f ' + env.WORK_DIR + 'Dockerfile' + ' --build-arg ' + buildArgs + ' -t ' + imageName + ' .'
                    }
                }
            }
        }

        stage('Push Docker Image') {
        
            when{
              branch 'master'
            }

            steps {
                container('docker') {

                    script {
                        def imageName = getImageName('imageName')
                        withDockerRegistry([credentialsId: 'suitelog', url: "https://docker.totvs.io"]) {
                            sh 'docker push ' + imageName
                        }
                    }
                }
            }
		}
        
        
        stage ('Deploy Interno') {

            environment {
                TAG_NAME = getImageName('tag')
            }

            when{
              branch 'master'
            }

            steps{

                dir ( './checklist-core/kubernetes/yaml' ) {
                    sh 'ls -l'
                    sh 'pwd'
                    sh 'echo $TAG_NAME'
                    sh '''
                        sed -i -e "s/TAG_NUMBER/$TAG_NAME/g" deployment-interno.yaml
                    '''

                    kubernetesDeploy(
                        kubeconfigId: "kubeconfig",
                        configs: "deployment-interno.yaml",
                        dockerCredentials: [[credentialsId: 'suitelog', url: "https://docker.totvs.io"]])
                }
            }
        }

        stage('Deploy on TKS') {

            environment {
                TAG_NAME = getImageName('tag')
            }
    
            when{
                branch 'master'
            }
    
            steps {
    
                dir ( './checklist-core/kubernetes/yaml' ) {
                    sh 'ls -l'
                    sh 'pwd'
                    sh 'echo $TAG_NAME'
                    sh '''
                        sed -i -e "s/TAG_NUMBER/$TAG_NAME/g" deployment-prod.yaml
                    '''
    
                    kubernetesDeploy(
                        kubeconfigId: "kubeconfig-tks",
                        configs: "deployment-prod.yaml",
                        secretNamespace: "logistica",
                        dockerCredentials: [[credentialsId: 'suitelog', url: "https://docker.totvs.io"]])
                 }
             }
        }
    }
}

def getInfoPOM() {

    def pom = readMavenPom file: env.WORK_DIR + 'pom.xml'

    return pom
}

def getImageName(kindService) {

    if ( kindService != 'tag') {

        def pom = getInfoPOM()
        def imageName = ""

        env.TAG_NAME = env.BUILD_NUMBER
        retServiceName = 'docker.totvs.io/sl_checklist/' + pom.groupId + '/' + pom.artifactId + ':' + env.BUILD_NUMBER

    } else {

    	env.TAG_NAME = env.BUILD_NUMBER
        retServiceName = env.BUILD_NUMBER

    }

    return retServiceName
}

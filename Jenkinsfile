pipeline {
	environment {
// 	  IS_RELEASE_TAG = sh(returnStdout: true, script: 'git tag --contains').trim().matches(/v\d{1,3}\.\d{1,3}\.\d{1,3}/)
      VERSION = sh(returnStdout: true, script: "./gradlew properties -q | grep version: | awk '{print \$2}'").trim()
      IS_SNAPSHOT = VERSION.endsWith('SNAPSHOT')
	}
	agent any
    tools {
        jdk 'jdk8'
    }
	stages {
		stage('Build') {
//             when {
//                 not {
//                     environment name:'IS_RELEASE_TAG', value:'true'
//                 }
//             }
            stages {
                stage('Compile') {
                    steps {
                        sh "echo ${scm.userRemoteConfigs.credentialsId}"
                        withGradle {
                	        sh './gradlew classes'
                	    }
                	}
                }
                stage('Validate') {
                    steps {
                        withGradle {
                            sh './gradlew check -x test'
                        }
                    }
                }
                stage('Unit Test') {
                    steps {
                        withGradle {
                            sh './gradlew test'
                        }
                	}
                }
                stage('Integration Test') {
                    steps {
                	    echo 'NO INTEGRATION TESTS - ./gradlew integration-test'
                	}
                }
                stage('Deploy Artifact') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'nexus',
                            usernameVariable: 'MVN_USER',
                            passwordVariable: 'MVN_PASSWORD')]) {
                                withGradle {
                                    sh './gradlew publish'
                                }
                        }
                	}
                }
            }
		}

		stage('Release') {
            when {
                allOf {
//                     not {
//                         environment name:'IS_RELEASE_TAG', value:'true'
//                     }
                    not {
                        environment name:'IS_SNAPSHOT', value:'true'
                    }
                }
            }
            stages {
                stage('Generate Release Notes') {
                    steps {
                        sh "echo 'test' > CHANGELOG.md"
                        sh "git add CHANGELOG.md"
                    }
                }
                stage('Tag Release') {
                    steps {
                        sh "git tag -a v${VERSION} -m \"Version ${VERSION}\""
                        withCredentials([usernamePassword(credentialsId: scm.userRemoteConfigs.credentialsId, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                            sh 'git config --local credential.helper "!f() { echo username=\\$GIT_USERNAME; echo password=\\$GIT_PASSWORD; }; f"'
                            sh 'git push --follow-tags'
                        }
                    }
                }
            }
		}

        stage('Increment SNAPSHOT Version') {
            when {
                environment name:'IS_RELEASE_TAG', value:'true'
            }
            steps {
                    echo 'Incrementing SNAPSHOT version'
                    echo 'git add pom.xml, commit, push'
            }
        }
    }
}
#!groovy
// 공유 라이브러리를 설정한다.
@Library('jenkins-lib') _

pipeline {
	agent any

	environment {
    	// 공유 라이브러리의 getGitRepoName() 함수를 호출한다. Repository 이름 파싱
		gitRepoName = getGitRepoName()
	}

	stages{
		stage('Checkout SCM') {
			steps {
				echo 'Checkout SCM!'
				// Jenkins 프로젝트에 설정된 Repository로 부터 현재 브랜치의 소스코드를 가져오는데 사용한다.
				checkout scm
			}
		}

		stage('Build') {
			steps {
				script {
                	// Maven Build Tool에 실행 권한을 부여한다.
					sh "chmod u+x ./mvnw"
                    sh "./mvnw clean package
				}
			}
		}

		stage('Deploy') {
			steps {
				echo 'Deploy is not yet implemented!'
			}
		}

		stage('SAST') {
			steps{
				script{
                	// SonarQube 작업을 호출한다.
					build job: 'SAST-SonarQube', parameters:[
						string(name: 'GIT_REPONAME', value: gitRepoName)
					], wait: false
				}
			}
		}
	}
}

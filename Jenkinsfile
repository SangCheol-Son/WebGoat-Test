#!groovy
pipeline {
    agent any
    
    parameters {
        string(name: 'GIT_REPONAME', description: 'Git Repository Name')
    }
    
    environment {
        // 프로젝트 정보만 설정
        PROJECT_KEY = "${params.GIT_REPONAME}"
        PROJECT_NAME = "${params.GIT_REPONAME}"
        // WebGoat-Test 저장소 URL
        GIT_REPO_URL = 'https://github.com/SangCheol-Son/WebGoat-Test.git'
    }
    
    stages {
        stage('Checkout WebGoat-Test Repository') {
            steps {
                script {
                    echo "=== WebGoat-Test 저장소 체크아웃 ==="
                    echo "Repository: ${GIT_REPO_URL}"
                    echo "Project Key: ${PROJECT_KEY}"
                    
                    // 기존 워크스페이스 정리
                    sh "rm -rf *"
                    
                    // WebGoat-Test 저장소 체크아웃
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                            url: GIT_REPO_URL,
                            credentialsId: 'jenkins_ssh_access_key_new'  // 기존 자격증명 사용
                        ]]
                    ])
                    
                    // 체크아웃된 파일 확인
                    sh "pwd && ls -la"
                }
            }
        }
        
        stage('Debug - Check Project Structure') {
            steps {
                script {
                    echo "=== 프로젝트 구조 확인 ==="
                    sh "pwd"
                    sh "ls -la"
                    
                    // Maven 프로젝트 구조 확인
                    if (fileExists('pom.xml')) {
                        echo "pom.xml 파일이 존재합니다."
                        sh "cat pom.xml | head -20"
                    } else {
                        echo "pom.xml 파일이 없습니다!"
                    }
                    
                    // src 디렉토리 확인
                    if (fileExists('src')) {
                        echo "src 디렉토리가 존재합니다."
                        sh "find src -type f -name '*.java' | head -10"
                    } else {
                        echo "src 디렉토리가 없습니다!"
                    }
                    
                    // mvnw 파일 확인
                    if (fileExists('mvnw')) {
                        echo "mvnw 파일이 존재합니다."
                    } else {
                        echo "mvnw 파일이 없습니다!"
                    }
                }
            }
        }
        
        stage('Build Project') {
            steps {
                script {
                    echo "=== 프로젝트 빌드 ==="
                    // Maven Build Tool에 실행 권한을 부여한다.
                    sh "chmod u+x ./mvnw"
                    // 컴파일만 수행 (애플리케이션 시작 제외)
                    sh "./mvnw clean compile test-compile"
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    echo "=== SonarQube 분석 시작 ==="
                    echo "Project Key: ${PROJECT_KEY}"
                    echo "Current directory: ${WORKSPACE}"
                    
                    // Jenkins 시스템 설정의 SAST-SonarQube 사용
                    withSonarQubeEnv('SAST-SonarQube') {
                        sh """
                            # 디버깅: 현재 디렉토리와 파일 확인
                            echo "=== SonarQube 분석 시작 ==="
                            pwd
                            ls -la
                            
                            # Maven을 사용한 SonarQube 분석 (verify 단계 제외)
                            ./mvnw clean compile test-compile sonar:sonar \
                                -Dsonar.projectKey=${PROJECT_KEY} \
                                -Dsonar.projectName=${PROJECT_NAME} \
                                -Dsonar.sources=src/main/java \
                                -Dsonar.tests=src/test/java \
                                -Dsonar.java.source=11 \
                                -Dsonar.sourceEncoding=UTF-8 \
                                -Dsonar.verbose=true \
                                -Dsonar.scm.provider=git \
                                -Dsonar.scm.forceReloadAll=true \
                                -Dsonar.java.binaries=target/classes \
                                -Dsonar.java.test.binaries=target/test-classes
                        """
                    }
                    
                    echo "SonarQube analysis completed for project: ${PROJECT_KEY}"
                }
            }
        }
        
        // Trivy 이미지 스캔 스테이지 추가
        stage('Trivy Image Scan') {
            steps {
                script {
                    echo "=== Trivy 이미지 스캔 시작 ==="
                    echo "Project: ${PROJECT_KEY}"
                    
                    // Dockerfile 확인
                    if (fileExists('Dockerfile')) {
                        echo "Dockerfile이 발견되었습니다."
                        
                        // Docker 이미지 빌드
                        sh "docker build -t ${PROJECT_KEY}:latest ."
                        
                        // Trivy로 이미지 스캔
                        sh """
                            echo "=== Trivy 스캔 실행 ==="
                            trivy image --severity HIGH,CRITICAL ${PROJECT_KEY}:latest
                            echo "=== Trivy 스캔 완료 ==="
                        """
                    } else {
                        echo "Dockerfile이 없습니다. Trivy 스캔을 건너뜁니다."
                        echo "Dockerfile을 생성하여 이미지 스캔을 활성화하세요."
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "SAST-SonarQube analysis completed for project: ${PROJECT_KEY}"
        }
        success {
            echo "SAST-SonarQube analysis succeeded!"
            echo "Check SonarQube dashboard for analysis results."
        }
        failure {
            echo "SAST-SonarQube analysis failed! Check the logs for details."
        }
    }
} 

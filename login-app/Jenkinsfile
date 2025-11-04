pipeline {
  agent any
  tools {
    jdk 'JDK17'
    maven 'Maven3'   // mvnw 쓰면 빼도 됨. steps에서 ./mvnw로 실행 가능
  }
  environment {
    APP_NAME   = 'login-app'                       // WAR 이름 & context path
    TOMCAT_URL = 'http://192.168.0.10:80'    // Tomcat 접근 URL
    CRED_ID    = 'Tomcat-Deployer'             // Jenkins 크리덴셜 ID
    USE_WRAPPER = 'true'                            // 'true'면 ./mvnw 사용
  }
  options {
    timestamps()
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build WAR') {
      steps {
        sh """
          if [ "${USE_WRAPPER}" = "true" ]; then
            chmod +x mvnw || true
            ./mvnw -B -DskipTests clean package
          else
            mvn -B -DskipTests clean package
          fi
        """
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        }
      }
    }
    stage('Deploy to Tomcat (Manager API)') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${CRED_ID}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh """
            # 기존 undeploy (없어도 OK)
            curl -sf -u "$USER:$PASS" "${TOMCAT_URL}/manager/text/undeploy?path=/${APP_NAME}" || true

            # 새 WAR deploy (update=true로 교체 가능)
            curl -sf -u "$USER:$PASS" \
              -T target/${APP_NAME}.war \
              "${TOMCAT_URL}/manager/text/deploy?path=/${APP_NAME}&update=true"
          """
        }
      }
    }
  }
  post {
    success { echo "배포 완료: ${TOMCAT_URL}/${APP_NAME}/" }
    failure { echo "빌드/배포 실패" }
  }
}

pipeline {
  agent any
  tools {
    maven 'maven3'   // 등록한 이름
    // jdk 'JAVA'  // JDK도 등록했다면 사용
  }
  stages {
    stage('Checkout'){ steps { checkout scm } }
    stage('Build'){
      steps { sh 'mvn -B clean package -DskipTests' }
    }
  }
}

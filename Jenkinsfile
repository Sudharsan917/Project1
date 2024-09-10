pipeline {
agent any
tools
{
jdk 'jdk22'
maven 'maven3'
}
environment
{
SCANNER_HOME = tool 'SonarQube_Scanner'
}
stages
{
stage('git checkout')
{
steps{
git branch: 'master',credentialsId: 'git-cred', url:'https://github.com/Sudharsan917/Project1.git'
}
}
stage('Compile')
{
steps{
sh "mvn compile"
}
}
stage('Test')
{
steps{
sh "mvn test"
}
}
stages('File System scan') {
steps {
sh "trivy fs --format table -o trivy-fs-report.html "
}
}
stage('SonarQube analsyis')
{
steps{
withSonarQubeEnv('sonar') {
  sh ''' $SCANNER_HOME/bin/sonar-scanner -
Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
-Dsonar.java.binaries=. '''

}
}
}
stage('Quality gate')
{
steps {
script {
waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'

}
}
stage('build')
{
steps {
sh "mvn package"
}
}
stage('publish to nexus')
{
steps{
withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk22', maven: 'maven3') {
    sh "mvn deploy"
}

}
}
stage('Build & tag docker image')
{
steps
{
script {
withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker')
 {
sh "docker build -t ganeshperumal007/boardshack:latest ."
}
}
}
}
stage('Docker image scan')
{
steps
{
sh "trivy image --format table -o trivy-image-report.html ganeshperumal007/boardshack:latest"
}
}
stage('push docker image')
{
steps{
script{
withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
sh "docker push ganeshperumal007/boardshack:latest"
}
}
}
stage('Deploy To Kubernetes')
{
steps{
withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '',
credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false,
serverUrl: 'https://172.31.8.146:6443') {
sh "kubectl apply -f deployment-service.yaml"
}
}
}
stage('verify the deployment')
{
steps {
withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '',
credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false,
serverUrl: 'https://172.31.8.146:6443') {
sh "kubectl get pods -n webapps"
sh "kubectl get svc -n webapps"
}
}
}
}
post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
            def body = """
<html>
<body>
<div style="border: 4px solid ${bannerColor}; padding: 10px;">
    <h2>${jobName} - Build ${buildNumber}</h2>
    <div style="background-color: ${bannerColor}; padding: 10px;">
        <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
    </div>
    <p>Check the <a href="${env.BUILD_URL}">console output</a>.</p>
</div>
</body>
</html>
"""

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'sudharsanyadav917@gmail.com', 
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}

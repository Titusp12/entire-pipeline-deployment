# entire-pipeline-deployment
node('maven'){
    def maven360home = tool name: 'maven360', type: 'maven'
    stage('Checkout'){
        git branch: 'batch2', credentialsId: 'titusp12_github', url: 'https://github.com/Titusp12/simple-java-maven-app.git'
    }

    stage('Building'){
        sh "$maven360home/bin/mvn clean package"
    }
    stage('Test'){
        sh "${maven360home}/bin/mvn clean test surefire-report:report-only"
        junit 'target/surefire-reports/*.xml'
    }
    stage('Archive'){
        archiveArtifacts 'target/surefire-reports/*'
    }
    stage('Publish'){
publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site/', reportFiles: 'surefire-report.html', reportName: 'HTMLReport', reportTitles: ''])
}
    stage('Deployment'){
sshagent(['mediator_target-mavenbuild']) {
    sh "scp -o StrictHostKeyChecking=no /home/ec2-user/jenkins_workspace/workspace/batch2/mavenbuild/target/my-app-1-RELEASE.jar deployuser@54.211.134.71:/home/deployuser"
}
    stage('storing in s3'){
withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws_key', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
    sh "aws s3 cp /home/ec2-user/jenkins_workspace/workspace/batch2/mavenbuild/target/my-app-1-RELEASE.jar s3://deployment345"
}
}
}
}

===============================================================================================
deploying to target server
 1) Opened 4th machine as mediator
 2) created a new user (deployuser)
 3) did ssh keygen for deployuser
 4) created deployuser inside target machine
 5) created a folder .ssh/authorized_keys w/ chmod 700 & 600 respectively
 6) added pub of 4th machine (deployuser) to authorized_keys of target machine's deployuser
 7) copied private key of mediator (deployuser) into credentials of Master Jenkins (ssh)
storage into S3:
 1) created a user with s3 full access and admin access
 2) downloaded AWS and EC2 plugin on jenkins
 3) using pipeline syntax "with credentials."
 4) provided new credentials w/ access key and secret key from S3 user.
 5) added that groovy statement into pipeline to copy that file into s3

pipeline {
    agent {
        label 'image-worker'
    }
    stages {
        stage('Build') {
            steps {
                echo 'Starting Build'
                sh 'mkdir -p build_output'
                sh 'zip -q -r build_output/build . -x distribution Jenkinsfile *.sh *.git* *.DS_Store* README.md build_output *.zip'
            }
        }
        stage('Publish') {
            steps {
                echo 'Starting Publish'
                def distribution = readFile('distribution').trim()
                sh """
                       # Get Distribution ID
                       dist_id=\$(aws cloudfront list-distributions --output text --query "DistributionList.Items[].[Aliases.Items[0],Id]" | egrep "^${distribution}" | cut -f 2)

                       echo "Distribution ID = $dist_id"
                       if [ -z \$dist_id ]; then
                         echo "Distribution: $distribution Not Found"
                         exit 1
                       fi

                       # Get S3 bucket, should match Origin ID
                       s3_bucket=\$(aws cloudfront get-distribution --id \$dist_id --output text --query "Distribution.DistributionConfig.Origins.Items[].Id")

                       echo "S3 bucket = \$s3_bucket"

                       aws s3 ls s3://\$s3_bucket >& /dev/null
                       if [ \$? -ne 0 ]; then
                         echo "S3 Bucket not found, check Cloudfront Distribution Origin ID"
                         exit 1
                       fi
                   """
                sh 'echo build_output'
                sh 'ls -lth build_output'
                sh """
                       cd build_output
                       unzip build.zip
                       ls -lth .
                   """
            }
        }
    }
}

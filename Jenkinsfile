def distribution
pipeline {
    agent {
        label 'image-worker'
    }
    stages {
        stage('Build') {
            steps {
                echo 'Starting Build'
                sh 'mkdir -p build_output'
                sh 'zip -q -r build_output/build . -x distribution Jenkinsfile *.sh *.git* *.DS_Store* README.md *build_output* *.zip'
            }
        }
        stage('Publish') {
            steps {
                echo 'Starting Publish'
                script { distribution = sh(script: "cat distribution", returnStdout: true).trim() }
                sh """
                       # Get Distribution ID
                       dist_id=\$(aws cloudfront list-distributions --output text --query "DistributionList.Items[].[Aliases.Items[0],Id]" | egrep "^${distribution}" | cut -f 2)

                       echo "Distribution ID = \$dist_id"
                       if [ -z \$dist_id ]; then
                         echo "Distribution: $distribution Not Found"
                         exit 1
                       fi

                       # Get S3 bucket, should match Origin ID
                       s3_bucket=\$(aws cloudfront get-distribution --id \$dist_id --output text --query "Distribution.DistributionConfig.Origins.Items[].Id")

                       echo "S3 bucket = \$s3_bucket"

                       aws s3 ls s3://\$s3_bucket > /dev/null
                       if [ \$? -ne 0 ]; then
                         echo "S3 Bucket not found, check Cloudfront Distribution Origin ID"
                         exit 1
                       fi
                       echo build_output
                       ls -lth build_output
                       cd build_output
                       unzip build.zip
                       ls -lth .
                       # remove contents of S3 bucket
                       echo "Clearing existing bucket contents: \$s3_bucket"
                       aws s3 rm s3://\$s3_bucket --recursive
                       if [ \$? -ne 0 ]; then
                         echo "Something went wrong clearing existing data from S3 bucket"
                         exit 1
                       fi
                       # Sync new dist to S3
                       echo "Copying new data to S3 bucket: \$s3_bucket"
                       aws s3 sync ./ --exclude 'build.zip' s3://\$s3_bucket

                       if [ \$? -ne 0 ]; then
                         echo "Something went wrong syncing new data to S3 bucket"
                         exit 1
                       fi

                       # Create invalidation
                       invalidation_id=\$(aws cloudfront create-invalidation --distribution-id \$dist_id --paths "/*" --output text --query "Invalidation.Id")

                       # Get invalidation status
                       printf "Distribution: \$distribution\nDistribution ID: \$dist_id\nInvalidation ID: \$invalidation_id\nInvalidation Path: \$invalidation_path\n"
                       status="InProgress"
                       while [ "\$status" != "Completed" ]; do
                         status=\$(aws cloudfront get-invalidation --distribution-id \$dist_id --id \$invalidation_id --output text --query "Invalidation.Status")
                         echo \$status
                         sleep 10
                       done
                   """
            }
        }
    }
}

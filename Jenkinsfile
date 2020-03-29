def ChangeIP
pipeline{
    agent any
    options{
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    stages{
        stage("Initialize"){
            steps{
                script{
                    deleteDir()
                }
            }
        }
        stage("Check IP again DNS "){
            steps{
                withCredentials(
                [[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {
                    script {
                        def HOSTED_ZONE_ID = input(
                            id: 'HOSTED_ZONE_ID', message: 'Enter Hoted Zone ID',
                            parameters: [

                                    string(name: 'Hosted-Zone-ID',
                                    description: 'Enter Hosted-Zone-ID'),
                            ])
                    sh """
                    aws route53 list-resource-record-sets --hosted-zone-id  ${HOSTED_ZONE_ID} --query "ResourceRecordSets[?Type == 'A']" --output table
                    """
                    }
                }
            }
        }
        stage("Check to change IP or Not"){
            steps {
                script {
                    ChangeIP = input(
                        id: 'Proceed', message: 'Do you want to change IP address..?', parameters: [
                            [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Proceed for new IP address..?']
                            ])
                    
                }
            }
        }
        stage("Change IP Address against DNS "){
            when {
                expression { ChangeIP == true }
            }
            steps{
                withCredentials(
                [[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {
                script {
                        def IP_Address = input(
                            id: 'IP_Address', message: 'Enter IP Address for DNS record',
                            parameters: [

                                    string(name: 'IP-Address',
                                    description: 'Enter IP Address'),
                            ])
                            
                    sh """
                    touch sample.json
                    cat > sample.json << EOF
{
            "Comment": "CREATE/DELETE/UPSERT a record ",
            "Changes": [{
            "Action": "UPSERT",
                        "ResourceRecordSet": {
                                    "Name": "sanjeev3d.tk",
                                    "Type": "A",
                                    "TTL": 300,
                                 "ResourceRecords": [{ "Value": "${IP_Address}"}]
}}]
}
                    """
                    sh """
                    aws route53 change-resource-record-sets --hosted-zone-id Z0304498114SVBULHG2DV --change-batch file://sample.json --output table
                    """
                }
            }
        }
    }
    }
}

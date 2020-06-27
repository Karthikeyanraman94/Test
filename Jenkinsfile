pipeline {
    agent any
    options {disableConcurrentBuilds()}
     stages {
 /*  
        environment {
        TFE_NAME = "app.terraform.io"
        TFE_URL = "https://app.terraform.io"
        TFE_ORGANIZATION = "VMware-Dynatrace"
        TFE_API_URL = "${TFE_URL}/api/v2"
        TFE_API_TOKEN = credentials("TerraformcloudCLI")
    }
   
  stages {
    stage('CleanWorkspace') {
            steps {
                cleanWs()
        }
    }  
    
   stage(' Test Terraform Commands') {
            steps {
                script {
                sh "cd /home/tfuser/terrafrom"
                   sh 'terraform --version'
                }
            }
        }
        
          stage('TFE Workstation list ') {
            steps {
                sh '''
                  curl \
                    --silent --show-error --fail \
                    --header "Authorization: Bearer $TFE_API_TOKEN" \
                    --header "Content-Type: application/vnd.api+json" \
                    ${TFE_API_URL}/organizations/${TFE_ORGANIZATION}/workspaces \
                    | jq -r \'.data[] | .attributes.name\'
                '''
            }
            
        }
            stage('TFE Set-up') {
            steps{
                sh '''
                     tee ~/.terraformrc > /dev/null <<EOF
                     credentials "${TFE_NAME}" {
                        token        = "${TFE_API_TOKEN}"
                     }
EOF
                '''
            }
        }
  */  
    stage('Initialze') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'edef1a18-a109-457f-986d-70019002202a', url: 'https://github.tools.sap/I501201/Terraform-Vsphere.git']]])
                sh "pwd"
                sh 'git ls-files | xargs -n 1 dirname | uniq | awk \'{print $1}\' ORS=\'\\n\' >>list.txt'
                sh "cat list.txt"
                sh "ls -ltr"
                script{
                liste = readFile 'list.txt'
                echo "Please chose the directory to terraform"
                env.Directory_SCOPE = input message: 'Please chose the directory to terraform ', ok: 'RUN!',
                        parameters: [choice(name: 'Directory_NAME', choices: "${liste}", description: 'Directory to build?')]
                    }
                }
        }


    stage('Terraform Init') {
      steps {
      dir("${env.Directory_SCOPE}"){
        sh "pwd"
        sh "ls -ltr"
        sh "terraform fmt -list=true -write=true -diff=true"
        sh "terraform init -input=false"
        sh "terraform validate"
       }
      }
    }
    stage('Terraform Plan') {
      steps {
        dir("${env.Directory_SCOPE}"){
        sh "pwd"
        sh "ls -ltr"
        sh "terraform plan -out=tfplan -input=false"
        sh " terraform graph -type=plan | dot -Tsvg > plan.svg"
        sh " terraform graph -type=plan-destroy | dot -Tsvg > plan-destroy.svg"
        sh " terraform graph -type=apply | dot -Tsvg > apply.svg"
        archiveArtifacts artifacts: '*.svg , tfplan', fingerprint: true
        }
      }
    }
    stage('Terraform Apply') {
      steps {
        dir("${env.Directory_SCOPE}"){
        sh "pwd"
        sh "ls -ltr"
        input 'Apply Plan'
        sh "terraform apply -input=false tfplan"
        sh "curl 172.15.254.41:8500/v1/kv/tf/state?raw > terraform.tfstate"
        archiveArtifacts artifacts: 'terraform.tfstate', fingerprint: true
        }
      }
    }
  }
}

pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-slave-pod
spec:
  containers:
  - name: jnlp
    image: 933277528084.dkr.ecr.ap-northeast-1.amazonaws.com/devops_prod_jp_cicdshare-jenkinsslave-jnlp6:3.0.1
    args:
    - \$(JENKINS_SECRET)
    - \$(JENKINS_NAME)
    volumeMounts:
    - name: docker-socket
      mountPath: /var/run/docker.sock
      readOnly: true     
    - name: efs-piplineshare-data
      subPath: awscli
      mountPath: /home/jenkins/awscli
    - name: efs-piplineshare-data
      subPath: kubectl
      mountPath: /home/jenkins/kubectl
    - name: efs-piplineshare-data
      subPath: pipline_script
      mountPath: /home/jenkins/pipline_script
    - name: jenkins-share
      mountPath: /home/jenkins/qy_wechat_sendshell/qy_wechat.sh
      readOnly: true
      subPath: qy_wechat.sh
    - name: efs-piplineshare-data
      subPath: node_modules
      mountPath: /home/jenkins/agent/workspace/web-intro-download-cicd
  - name: node
    image: node:14.18.1-alpine3.14
    command:
    - cat
    tty: true
    volumeMounts: 
    - name: efs-piplineshare-data
      subPath: node_modules/web-intro-download
      mountPath: /home/jenkins/agent/workspace/web-intro-download
  volumes:
  - name: jenkins-share
    configMap:
      name: jenkins-share
  - name: efs-piplineshare-data
    persistentVolumeClaim:
      claimName: efs-piplineshare
  - name: docker-socket
    hostPath:
      path: /var/run/docker.sock
  resources:
    requests:
      memory: "500Mi"
      cpu: "1000m"
"""      
    }
  }

  triggers {
    GenericTrigger(
     genericVariables: [
      [key: 'ref', value: '$.ref']
     ],
     causeString: 'Triggered on $ref',
     token: 'whalefin_alltest_jp_web-intro-download',
     printContributedVariables: true,
     printPostContent: true,
     silentResponse: false,
     regexpFilterText: '$ref',
     regexpFilterExpression: 'refs/heads/' + BRANCH_NAME
    )
  }


environment 
{
    AlltestimageName='whalefin_alltest_jp_web-intro-download'
    ProdimageName='whalefin_prod_jp_web-intro-download'
    awsregion='ap-northeast-1'                           //--------------需要变更的配置行
    aws_ecr_path='933277528084.dkr.ecr.ap-northeast-1.amazonaws.com'                  //--------------需要变更的配置行
    AWS_EKS_TEST_KEY = credentials('aws-eks-app-aceup-test') 
    prodenv_eks_accesskey = credentials('AWS-JP-WHALEFIN-EKS-PROD')
    ENV_PROD_BRANCH='prod'
    ENV_TEST_BRANCH='k8s'
    ENV_DEV_BRANCH='dev'
    ENV_UAT_BRANCH='uat'
    ENV_SIT_BRANCH='sit'
    BUILD_ENV_DEV='dev'
    BUILD_ENV_UAT='uat'
    BUILD_ENV_SIT='sit'
    BUILD_ENV_TEST='test'
    BUILD_ENV_PROD='prod'
    // aws_eks_test_cluster_name='AWS-JP-APP-ACEUP-EKS-TEST'
    // aws_eks_prod_cluster_name='AWS-JP-APP-ACEUP-EKS-PROD'
    aws_eks_test_cluster_name="AWS-JP-APP-ACEUP-EKS-TEST"
    aws_eks_prod_cluster_name="AWS-JP-WHALEFIN-EKS-PROD"
    k8s_deployment_yaml_filename='whalefin-web-intro-download-deployment.yaml'
    k8s_deployment_prod_yaml_filename='whalefin-web-intro-download-prod-deployment.yaml'
    k8s_service_yaml_filename='whalefin-web-intro-download-service.yaml'
    k8s_worlkload_type='deployment'
    k8s_worlkload_name='whalefin-h5-web-intro-download'
    prod_namespacename='whalefin-front'
    test_namespacename='whalefin-test'
    dev_namespacename='whalefin-dev'
    uat_namespacename='whalefin-uat'
    sit_namespacename='whalefin-sit'
    local_awscli_path='/home/jenkins/awscli/aws/dist/aws'
    local_kubectl_path='/home/jenkins/kubectl'
    imageTag = sh(returnStdout: true,script: 'git describe --tags --always').trim()      
    qy_wechat = '/home/jenkins/qy_wechat_sendshell/qy_wechat.sh'
    G_aws_devops_wechat_token = credentials('G_aws_devops_wechat_token')
    jenkins_k8s_deployment_health_check_filename='/home/jenkins/pipline_script/jenkins_k8s_deployment_health_check.sh'
    jenkins_k8s_cdn_filename='/home/jenkins/pipline_script/akamai-purge'
    jenkins_k8s_cdn_prod_filename='/home/jenkins/pipline_script/akamai-purge-prod'
    jenkinsslave_homedir='/root'
    prodenv_kube_config = credentials('AWS-JP-WHALEFIN-EKS-PROD')
    newtestenv_eks_accesskey = credentials('AWS-JP-WHALEFIN-EKS-TEST')
    test_Affinity='AWS-JP-WHALEFIN-EKS-TEST-FRONTEND01'
    prod_Affinity='AWS-JP-WHALEFIN-EKS-PROD-FRONT01'
    app_Affinity='AWS-JP-APP-ACEUP-EKS-TEST-nodegroup4'
}


stages 
{  
  stage('build node_modules') {
    steps {
          container('node') {
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>init DB config<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
			dir("nginx") {
             git branch: 'master', credentialsId: 'G_aws_devops_gitlab_ssh_PrivateKey', url: 'git@git.amberainsider.com:web/h5-template.git'
            }
            script {
               if (env.BRANCH_NAME == "${env.ENV_DEV_BRANCH}") {
                  sh """
                  echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                  yarn install
                  yarn build:${env.ENV_DEV_BRANCH}
                  """
               } else if (env.BRANCH_NAME == "${env.ENV_UAT_BRANCH}") {
                  sh """
                  echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                  yarn install
                  yarn build:${env.ENV_UAT_BRANCH}
                  """
               }else if (env.BRANCH_NAME == "${env.ENV_SIT_BRANCH}") {
                  sh """
                  echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                  yarn install
                  yarn build:${env.ENV_SIT_BRANCH}
                  """
               }else if (env.BRANCH_NAME == "${env.ENV_PROD_BRANCH}") {
                  sh """
                  echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                  yarn install
                  yarn build
                  """
               }else {
                  sh """
                  echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                  yarn install
                  yarn build:${env.ENV_TEST_BRANCH}
                  """
               } 
            }    
      }
    }
  }
  stage('build and push image') {
    steps {
          container('jnlp') {
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>build and push image<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            sh """
            ${env.local_awscli_path} ecr get-login-password --region ${env.awsregion} | docker login --username AWS --password-stdin ${env.aws_ecr_path}
            """
            script {
            	if (env.BRANCH_NAME == "${env.ENV_PROD_BRANCH}") {
                        sh """
                        echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                        #cd /home/jenkins/agent/workspace/web-intro-download-cicd/web-intro-download/master
			cp -rf nginx/devops-cicd/conf  devops-cicd/
			cp -rf dist devops-cicd/
                        cd devops-cicd
                        sed -i 's/<BUILD_ENV>/${env.BUILD_ENV_PROD}/'  Dockerfile_prod
			docker build -t ${env.ProdimageName}:${env.imageTag}-${env.BRANCH_NAME} -f Dockerfile_prod --network host .
                        docker tag ${env.ProdimageName}:${env.imageTag}-${env.BRANCH_NAME}  ${env.aws_ecr_path}/${env.ProdimageName}:${env.imageTag}-${env.BRANCH_NAME}
                        docker push ${env.aws_ecr_path}/${env.ProdimageName}:${env.imageTag}-${env.BRANCH_NAME}-${env.BRANCH_NAME}-${env.BRANCH_NAME}
                        """
                 } else if (env.BRANCH_NAME == "${env.ENV_DEV_BRANCH}"){
                        sh """
                        echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                        #cd /home/jenkins/agent/workspace/web-intro-download-cicd/web-intro-download/dev
			cp -rf nginx/devops-cicd/conf  devops-cicd/
			cp -rf dist devops-cicd/
                        cd devops-cicd
                        sed -i 's/<BUILD_ENV>/${env.BUILD_ENV_DEV}/' Dockerfile
                        docker build -t ${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}-${env.BRANCH_NAME}-${env.BRANCH_NAME} -f Dockerfile --network host .
                        docker tag ${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME} ${env.aws_ecr_path}/${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}
                        docker push ${env.aws_ecr_path}/${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}-${env.BRANCH_NAME}-${env.BRANCH_NAME}
                        """      
  
                } else if (env.BRANCH_NAME == "${env.ENV_SIT_BRANCH}"){
                        sh """
                        echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                        #cd /home/jenkins/agent/workspace/web-intro-download-cicd/web-intro-download/sit
                        cp -rf nginx/devops-cicd/conf  devops-cicd/
                        cp -rf dist devops-cicd/
                        cd devops-cicd
                        sed -i 's/<BUILD_ENV>/${env.BUILD_ENV_SIT}/' Dockerfile
                        docker build -t ${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME} -f Dockerfile --network host .
                        docker tag ${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME} ${env.aws_ecr_path}/${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}
                        docker push ${env.aws_ecr_path}/${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}
                        """      
  
                }else if (env.BRANCH_NAME == "${env.ENV_UAT_BRANCH}"){
                        sh """
                        echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                        #cd /home/jenkins/agent/workspace/web-intro-download-cicd/web-intro-download/uat
                        cp -rf nginx/devops-cicd/conf  devops-cicd/
                        cp -rf dist devops-cicd/
                        cd devops-cicd
                        sed -i 's/<BUILD_ENV>/${env.BUILD_ENV_UAT}/' Dockerfile
                        docker build -t ${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME} -f Dockerfile --network host .
                        docker tag ${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME} ${env.aws_ecr_path}/${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}
                        docker push ${env.aws_ecr_path}/${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}
                        """      
  
                }else {
                        sh """
                        echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
                        #cd /home/jenkins/agent/workspace/web-intro-download-cicd/web-intro-download/test
                        cp -rf nginx/devops-cicd/conf  devops-cicd/
                        cp -rf dist devops-cicd/
                        cd devops-cicd
                        sed -i 's/<BUILD_ENV>/${env.BUILD_ENV_TEST}/'  Dockerfile
                        docker build -t ${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME} -f Dockerfile --network host .
                        docker tag ${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME} ${env.aws_ecr_path}/${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}
                        docker push ${env.aws_ecr_path}/${env.AlltestimageName}:${env.imageTag}-${env.BRANCH_NAME}
                        """
            }
         } 
      }
    }
  }
  stage('change YAML and Deploy Prod') {
     when {
               branch "${env.ENV_PROD_BRANCH}"
          }
    steps {
          container('jnlp') {
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            echo ""
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>Change YAML File Stage<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            sh "sed -i 's/<BUILD_REGION>/${env.awsregion}/' devops-cicd/${env.k8s_deployment_prod_yaml_filename}"
            sh "sed -i 's/<BUILD_IMAGE>/${env.ProdimageName}/' devops-cicd/${env.k8s_deployment_prod_yaml_filename}"
            sh "sed -i 's/<BUILD_TAG>/${env.imageTag}-${env.BRANCH_NAME}/g' devops-cicd/${env.k8s_deployment_prod_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.prod_namespacename}/' devops-cicd/${env.k8s_deployment_prod_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.prod_namespacename}/' devops-cicd/${env.k8s_service_yaml_filename}"
	        echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>Deploy Stage<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            sh "mkdir -p ${env.jenkinsslave_homedir}/.kube/"
            sh "cp -rp ${prodenv_eks_accesskey}  ${env.jenkinsslave_homedir}/.kube/config"
            timeout(time: 5, unit: 'MINUTES') {
            script {
               input(
                 message: 'Should we continue?',
                 ok: "YES"
                )
              }
            }
            sh " ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_deployment_prod_yaml_filename}"
            sh " ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_service_yaml_filename}"
            sh "sleep 5"
            sh "${env.jenkins_k8s_deployment_health_check_filename} ${env.local_kubectl_path} ${env.k8s_worlkload_type} ${env.k8s_worlkload_name} ${env.prod_namespacename}  ${env.ProdimageName} ${env.imageTag}-${env.BRANCH_NAME}"
            sh "sleep 20"
            sh "echo start cdn h5.whalefin.com"
            sh "${env.jenkins_k8s_cdn_prod_filename} -cpcode 1257244"
            sh "echo reload cdn end"

       }
     }
   }
   stage('change YAML and Deploy dev') {
     when {
            branch "${env.ENV_DEV_BRANCH}"
          }
     steps {
          container('jnlp') {
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            echo ""
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>Change YAML File Stage<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            sh "sed -i 's/<BUILD_REGION>/${env.awsregion}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_IMAGE>/${env.AlltestimageName}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_TAG>/${env.imageTag}-${env.BRANCH_NAME}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.dev_namespacename}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.dev_namespacename}/' devops-cicd/${env.k8s_service_yaml_filename}"
            sh "sed -ie 's/AFFINITY/${env.test_Affinity}/g' devops-cicd/${env.k8s_deployment_yaml_filename}"
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>Deploy Stage>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
            sh """
            mkdir -p ${env.jenkinsslave_homedir}/.kube/
            cp -rp ${newtestenv_eks_accesskey}  ${env.jenkinsslave_homedir}/.kube/config
            ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_deployment_yaml_filename}
            ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_service_yaml_filename}
            sleep 5
            ${env.jenkins_k8s_deployment_health_check_filename} ${env.local_kubectl_path} ${env.k8s_worlkload_type} ${env.k8s_worlkload_name} ${env.dev_namespacename}  ${env.AlltestimageName} ${env.imageTag}-${env.BRANCH_NAME}
            """ 
       }
     }
   }
   stage('change YAML and Deploy sit') {
     when {
            branch "${env.ENV_SIT_BRANCH}"
          }
     steps {
          container('jnlp') {
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            echo ""
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>Change YAML File Stage<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            sh "sed -i 's/<BUILD_REGION>/${env.awsregion}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_IMAGE>/${env.AlltestimageName}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_TAG>/${env.imageTag}-${env.BRANCH_NAME}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.sit_namespacename}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.sit_namespacename}/' devops-cicd/${env.k8s_service_yaml_filename}"
            sh "sed -ie 's/AFFINITY/${env.test_Affinity}/g' devops-cicd/${env.k8s_deployment_yaml_filename}"
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>Deploy Stage>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
            sh """
            mkdir -p ${env.jenkinsslave_homedir}/.kube/
            cp -rp ${newtestenv_eks_accesskey}  ${env.jenkinsslave_homedir}/.kube/config
            ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_deployment_yaml_filename}
            ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_service_yaml_filename}
            sleep 5
            ${env.jenkins_k8s_deployment_health_check_filename} ${env.local_kubectl_path} ${env.k8s_worlkload_type} ${env.k8s_worlkload_name} ${env.sit_namespacename}  ${env.AlltestimageName} ${env.imageTag}-${env.BRANCH_NAME}
            """
       }
     }
   }
   stage('change YAML and Deploy UAT') {
     when {
            branch "${env.ENV_UAT_BRANCH}"
          }
     steps {
          container('jnlp') {
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            echo ""
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>Change YAML File Stage<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            sh "sed -i 's/<BUILD_REGION>/${env.awsregion}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_IMAGE>/${env.AlltestimageName}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_TAG>/${env.imageTag}-${env.BRANCH_NAME}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.uat_namespacename}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.uat_namespacename}/' devops-cicd/${env.k8s_service_yaml_filename}"
            sh "sed -ie 's/AFFINITY/${env.app_Affinity}/g' ${env.k8s_deployment_yaml_filename}"
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>Deploy Stage>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
            sh """
            mkdir -p ~/.aws
            touch ~/.aws/credentials
            touch ~/.aws/config
            echo '[default]' >> ~/.aws/credentials
            echo '[default]' >> ~/.aws/config
            echo "aws_access_key_id = ${env.AWS_EKS_TEST_KEY_USR}" >> ~/.aws/credentials
            echo "aws_secret_access_key = ${env.AWS_EKS_TEST_KEY_PSW}" >> ~/.aws/credentials
            echo "region = ${env.awsregion}" >> ~/.aws/config
            echo "output = json" >> ~/.aws/config
            /usr/local/bin/aws eks --region ${env.awsregion} update-kubeconfig --name ${env.aws_eks_test_cluster_name}
            ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_deployment_yaml_filename}
            ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_service_yaml_filename}
            sleep 5
            ${env.jenkins_k8s_deployment_health_check_filename} ${env.local_kubectl_path} ${env.k8s_worlkload_type} ${env.k8s_worlkload_name} ${env.uat_namespacename}  ${env.AlltestimageName} ${env.imageTag}-${env.BRANCH_NAME}
            ${env.jenkins_k8s_cdn_filename} -cpcode 1257266
            """
       }
     }
   }
   stage('change YAML and Deploy test') {
     when {
            branch "${env.ENV_TEST_BRANCH}"
          }
     steps {
          container('jnlp') {
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>This is a deploy step to ${env.BRANCH_NAME}<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            echo ""
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>Change YAML File Stage<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"
            sh "sed -i 's/<BUILD_REGION>/${env.awsregion}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_IMAGE>/${env.AlltestimageName}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_TAG>/${env.imageTag}-${env.BRANCH_NAME}-${env.BRANCH_NAME}-${env.BRANCH_NAME}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.test_namespacename}/' devops-cicd/${env.k8s_deployment_yaml_filename}"
            sh "sed -i 's/<BUILD_NAMESPACE>/${env.test_namespacename}/' devops-cicd/${env.k8s_service_yaml_filename}"
            echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>Deploy Stage>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
            sh """
            mkdir -p ~/.aws
            touch ~/.aws/credentials
            touch ~/.aws/config
            echo '[default]' >> ~/.aws/credentials
            echo '[default]' >> ~/.aws/config
            echo "aws_access_key_id = ${env.AWS_EKS_TEST_KEY_USR}" >> ~/.aws/credentials
            echo "aws_secret_access_key = ${env.AWS_EKS_TEST_KEY_PSW}" >> ~/.aws/credentials
            echo "region = ${env.awsregion}" >> ~/.aws/config
            echo "output = json" >> ~/.aws/config
            /usr/local/bin/aws eks --region ${env.awsregion} update-kubeconfig --name ${env.aws_eks_test_cluster_name}
            ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_deployment_yaml_filename}
            ${env.local_kubectl_path} apply -f devops-cicd/${env.k8s_service_yaml_filename}
            sleep 5
            """
     }
   }
 }
}
post { 
	always{
	//always部分 pipeline运行结果为任何状态都运行
          echo 'post stage'
      }
      success {
          //当此Pipeline成功时打印消息
          echo 'success'
          //sh "sh  ${env.qy_wechat}  ${NODE_NAME} ${env.job_name} ${BUILD_URL}  ${GIT_COMMIT}  success  ${env.G_aws_devops_wechat_token}"
          // 可以加上邮件、钉钉通知
      }
      failure {
          echo 'failure'
          // 可以加上邮件、钉钉通知
          //sh "sh  ${env.qy_wechat}  ${NODE_NAME} ${env.job_name} ${BUILD_URL}  ${GIT_COMMIT}  failure ${env.G_aws_devops_wechat_token}"
      }
	unstable {
          //当此Pipeline 为不稳定时打印消息
          echo 'unstable'	
          //sh "sh  ${env.qy_wechat}  ${NODE_NAME} ${env.job_name} ${BUILD_URL}  ${GIT_COMMIT}  unstable ${env.G_aws_devops_wechat_token}"
	}
	aborted {
	    //当此Pipeline 终止时打印消息
          echo 'aborted'	
          //sh "sh  ${env.qy_wechat}  ${NODE_NAME} ${env.job_name} ${BUILD_URL}  ${GIT_COMMIT}  aborted ${env.G_aws_devops_wechat_token}"
	}
	changed {
	    //当pipeline的状态与上一次build状态不同时打印消息
          echo 'changed'		
          //sh "sh  ${env.qy_wechat}  ${NODE_NAME} ${env.job_name} ${BUILD_URL}  ${GIT_COMMIT}  changed ${env.G_aws_devops_wechat_token}"
	}        
  }
}


pipeline {
   agent { node { label 'docker_node'  }}
   stages {
     
      stage('blue-green-deployment') {
          steps{  script{          sh '''
          set +e

         /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config get ns
         /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config get ns | grep  ${NAME_SPACE}
         if [ $? -eq 0 ]
         then
           echo "found namespace ${NAME_SPACE}  step 2  "
            /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config get deployment -n ${NAME_SPACE} | grep ${APP_NAME}
            if [ $? -eq 0 ]
            then
                echo "Found- ${APP_NAME}  step 3 "
                echo "Stage deployment - ${VERSION}"
                
                CLUSTER_NAME_STAGE=${CLUSTER_NAME}-stage
                VIRTUAL_NAME_STAGE=${VIRTUAL_NAME}-stage
                DNS_NAME_STAGE=${DNS_NAME}-stage
                
                sed -i 's/'${CLUSTER_NAME}'/'${CLUSTER_NAME_STAGE}'/g' k8s/blue-green/2blue-green-deployment.yaml
                sed -i 's/'${VIRTUAL_NAME}'/'${VIRTUAL_NAME_STAGE}'/g' k8s/blue-green/2blue-green-deployment.yaml
                sed -i 's/'${DNS_NAME}'/'${DNS_NAME_STAGE}'/g' k8s/blue-green/2blue-green-deployment.yaml

                sed -i 's/'${CLUSTER_NAME}'/'${CLUSTER_NAME_STAGE}'/g' k8s/blue-green/3blue-green-cluster-service.yaml
                sed -i 's/'${VIRTUAL_NAME}'/'${VIRTUAL_NAME_STAGE}'/g' k8s/blue-green/3blue-green-cluster-service.yaml
                sed -i 's/'${DNS_NAME}'/'${DNS_NAME_STAGE}'/g' k8s/blue-green/3blue-green-cluster-service.yaml

                sed -i 's/'${CLUSTER_NAME}'/'${CLUSTER_NAME_STAGE}'/g' k8s/blue-green/4blue-green-virtual-service.yaml
                sed -i 's/'${VIRTUAL_NAME}'/'${VIRTUAL_NAME_STAGE}'/g' k8s/blue-green/4blue-green-virtual-service.yaml
                sed -i 's/'${DNS_NAME}'/'${DNS_NAME_STAGE}'/g' k8s/blue-green/4blue-green-virtual-service.yaml

                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/2blue-green-deployment.yaml
                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/3blue-green-cluster-service.yaml
                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/4blue-green-virtual-service.yaml
                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config get vs | grep  ${DNS_NAME}
                cat k8s/blue-green/template_blue_green.yaml
                
            else 
                echo "Not found  - ${APP_NAME} step 4 "
                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/1blue-green-namespace.yaml
                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/2blue-green-deployment.yaml
                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/3blue-green-cluster-service.yaml
                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/4blue-green-virtual-service.yaml
                /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config get vs | grep  ${DNS_NAME}
            fi
         else 
            echo "not found Namespace ${NAME_SPACE} - create new namespace  - step 1 "
            /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/1blue-green-namespace.yaml
            /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config get ns | grep  ${NAME_SPACE}
           if [ $? -eq 0 ]
           then
            /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/2blue-green-deployment.yaml
            /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/3blue-green-cluster-service.yaml
            /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/4blue-green-virtual-service.yaml
            /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config get vs | grep  ${DNS_NAME}
           fi
         fi
         
       
        #echo "latest"
        #echo "Output yaml file"
        #sh " cat k8s/blue-green/template_blue_green.yaml"
         
       '''
       
              }
      }
    } 
    
    stage ('WaitForTestingStage') {
            input {       message "Ready to SWAP Live Listener?"            
                         ok "Yes, go ahead."      }
            steps {              echo "Moving on to perform SWAP ..................."            }            
        }
    
    stage('switch traffic blue to green') {
          steps{ 
              script {  
                   sh ''' 

                        CLUSTER_NAME_STAGE=${CLUSTER_NAME}-stage
                        echo ${CLUSTER_NAME_STAGE}
                        echo ${CLUSTER_NAME}
                         sed -i 's/'${CLUSTER_NAME_STAGE}'/'${CLUSTER_NAME}'/g' k8s/blue-green/3blue-green-cluster-service.yaml
                         cat k8s/blue-green/3blue-green-cluster-service.yaml
                         
                         /home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config apply -f k8s/blue-green/3blue-green-cluster-service.yaml
                    '''   
            }  }
                                } 

    stage ('WaitForConfirmation') {
            input {       message "Delete OLD deployment ?"           
                          ok "Yes, go ahead."      }
            steps {              echo "Moving on to perform delete old deployment  ..................."            }            
        }
    
    stage('Delete old deployment') {
          steps{              
            sh '/home/centos/linux-amd64/kubectl --kubeconfig /home/centos/linux-amd64/kube.config get deployment -n ${NAME_SPACE}'
                }
                                } 
  
  }
}

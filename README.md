

 ![Screenshot argo ](images/argo-pods.png?raw=true "POST")

 ![Screenshot argo ](images/argo-workflow.png?raw=true "POST")


1. instalar minikube
    https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download
 
2. Instalar Helm
    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
    helm version

3. Instalar Argo Workflow
    helm repo add argo https://argoproj.github.io/argo-helm
    helm repo update
    # Instalamos y creamos namespace para argo
    kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml

        --damos permisos a argo
        kubectl create rolebinding argo-configmap-binding \
            --clusterrole=view \
            --serviceaccount=argo:argo \
            --namespace=argo
        -- validar los pods 
            kubectl get pods -n argo  
            kubectl get svc -n argo

    # Para acceder a la interfaz web de Argo Workflow:
        kubectl -n argo port-forward deployment/argo-server 2746:2746

    # Verifica que los pods de Argo están corriendo:


4. Construcción de la imagen Docker y deploy en AKS
    
   # Descargar la última versión de Argo CLI
        
   # Creamos pvc para compartir los archivos

        kubectl apply -f pvc/pvc.yaml -n argo
        kubectl get pvc -n argo

   # creamos el secreto para credenciales a github
   
        kubectl create secret generic docker-secret \
        --from-literal=username=tu_usuario \
        --from-literal=password=tu_contraseña \
        -n argo

   # Permisos a argo

        kubectl create clusterrolebinding argo-cluster-admin \
        --clusterrole=cluster-admin \
        --serviceaccount=argo:argo
  
   # Creamos secreto para la utenticacion con docker hub
  
        kubectl create secret generic docker-hub-secret \
        --namespace=argo \
        --from-literal=username=lgomezs \
        --from-literal=password=PASSWORD

        validate:
        echo "PASSWORD" | docker login -u "lgomezs" --password-stdin https://index.docker.io/v1/

    # creamos namespace para el despliegue de aplicacion
       kubectl create namespace application

      ## asiganmos permisos al namespace 

           kubectl apply -f roles/argo-role.yaml
           kubectl apply -f roles/argo-rolebinding.yaml

   # Ejecutamos 

        argo submit deploy-workflow.yaml -n argo

        argo list -n argo

        argo logs <nombre-del-workflow> -n argo

        argo delete --all -n argo

        
  
 # Exponer Argo Workflows con ngrok

    sudo snap install ngrok
    ngrok http 2746

 # Configurar GitHub Secrets

   Crear el secreto KUBECONFIG_BASE64, con el kubeconfig.

    cat ~/.kube/config | base64 
     
  Para entorno de prueba se configuro URL de argo con ngrok -> crear variable variable NGROK_URL
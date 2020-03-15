# k8s-easyappointments
To deploy EasyAppoinments web app onto a kubernetes cluster:
 a). clone this repo ("https://github.com/putical/k8s-easyappointments.git") on a kubernetes cluster master node
 b). run command 'kubectl apply -f ./k8s-easyappointments/' to deploy the app

Note: this method is to deploy 'EasyAppointments' app for a sample client, after run the above command, below items will be deployed onto the kubernetes cluster: (yaml files are templates)
   1. two pods (one for application, the other for DB instance)
   2. two deployments objects (one for application, the other for DB instance)
   3. two services objects (one for application, the other for DB instance)
   4. a ConfigMap object (stores the environment variables for both containers, for this example, there are sensitive information like         DB passwords stored in the file which is not secure. Ideally, sensitive information should be saved into 'secret' objects' and           pods decrypte those credentials when start up from mounted volumes
   5. two persistent volumes (one for application, the other for DB instance)
   6. two persistent volume claims (one for application persistent volume, the other for DB persistent volume)
   
 
 A possible solution to automate application deployment for different clients:
 To differentiate deployments between different clients, names of pods/deployment/services/pvc/pv need to be different, this can be       achieved by changing the name attribute under 'metadata' in each yaml file and host mount paths for App and DB persistent volumes. 
  
 To automate it, a simple bash script with a series of 'sed' commands can be used to do content search & replace against those yaml template files. When there are new clients to be added, run the script with the client name as argument. 
 
example deployments outputs would be like this: 
*********************************************************************************************************************************
sgao@master:~/k8s-easyappointments$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
application-clientA-6499ff5dcb-7kc6z   1/1     Running   0          28m
database-clientA-59c4678566-bd46b      1/1     Running   0          28m
application-clientB-3829ba4fad-8tyuc   1/1     Running   0          50m
database-clientB-4323gi4ihy-hn94c      1/1     Running   0          50m

sgao@master:~/k8s-easyappointments$ kubectl get deployments
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
application-clientA   1         1         1            1           28m
database-clientA      1         1         1            1           28m
application-clientB   1         1         1            1           50m
database-clientB      1         1         1            1           50m

sgao@master:~/k8s-easyappointments$ kubectl get svc
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
application-clientA   ClusterIP   10.104.205.100   <none>        80/TCP      28m
database-clientA      ClusterIP   None             <none>        55555/TCP   28m
application-clientA   ClusterIP   10.104.205.101   <none>        80/TCP      50m
database-clientA      ClusterIP   None             <none>        55555/TCP   50m
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP     27h
  
sgao@master:~/k8s-easyappointments$ kubectl get pv
NAME                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                               STORAGECLASS   REASON   AGE
application-ClientA-pv   1Gi        RWO            Retain       Bound    default/easy-appointments-data                              29m
db-ClientA-pv            1Gi        RWO            Retain       Bound    default/easy-appointments-storage                           29m
application-ClientB-pv   1Gi        RWO            Retain       Bound    default/easy-appointments-data                              51m
db-ClientB-pv            1Gi        RWO            Retain       Bound    default/easy-appointments-storage                           51m
************************************************************************************************************************************


    

# kubeDock-IBM


```

alias k=kubectl
  287  k get nodes
  288  k get pods
  289  clear
  290  k create deploy dep1 --image httpd --replicas 5
  291  k get pods
  292  k get pods -o wide
  293  clear
  294  k run nginx --image nginx
  295  k get pods
  296  k get svc
  297  k delete svc ext-svc
  298  clear
  299  k get svc
  300  clear
  301  k get pods
  302  k expose pod nginx --name nginxsvc --port 80 --target-port 80 --type NodePort 
  303  k get svc
  304  k get pods -o wide
  305  curl 172.31.54.113
  306  curl 172.31.54.113:30043
  307  clear
  308  k get all
  309  k expose deployment dep1 --name dep1svc --target-port 80 --port 80 --type NodePort
  310  k get svc
  311  k describe svc nginxsvc
  312  k get pods --selector run=nginx
  313  k get pods -o wide
  314  k get svc
  315  k describe svc dep1svc
  316  k get pods --select app=dep1
  317  k get pods --selector app=dep1
  318  k get pods -o wide
  319  k get svc
  320  k scale deploy --replicas 1
  321  k scale deploy dep1--replicas 1
  322  k scale deploy dep1 --replicas 1
  323  k get pods -o wide
  324  k expose deployment dep1 --name dep2svc --target-port 80 --port 80 --type ClusterIp
  325  k expose deployment dep1 --name dep2svc --target-port 80 --port 80 --type ClusterIP
  326  clear
  327  k get svc
  328  k describe svc dep2svc
  329* 
  330  k get svc
  331  k get ep
  332  k get pods
  333  k scale deploy dep1 --replicas 5
  334  k describe svc dep2svc

```



CANARY AND BLUE GREEN :
```

root@kube-master:~# cat blue-nginx-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: rk1
  template:
    metadata:
      labels:
        app: rk1
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80



root@kube-master:~# cat green-httpd.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rk1
  template:
    metadata:
      labels:
        app: rk1
    spec:
      containers:
      - name: httpd
        image: httpd
        ports:
        - containerPort: 80


root@kube-master:~# cat service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ext-svc
spec:
  type: NodePort
  selector:
    app: rk1
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)



HISTORY :

k create -f blue-nginx-deploy.yaml
  453  k get pods
  454  clear
  455  vi service.yaml
  456  cat blue-nginx-deploy.yaml
  457  vi service.yaml
  458  netstat -tulnp
  459  clear
  460  ls
  461  k get pods
  462  k get svc
  463  k describe svc kubernetes
  464  k get pods -A
  465  k get pods -A -o wide
  466  k get svc
  467  k describe svc kubernetes
  468  clear
  469  ls
  470  k get svc
  471  k get deploy
  472  k create -f service.yaml
  473  k get all
  474  cat service.yaml
  475  clear
  476  k get svc
  477  k get deploy -o wide
  478  k get pods -o wide
  479  clear
  480  ls
  481  cat blue-nginx-deploy.yaml
  482  clear
  483  vi green-httpd.yaml
  484  k get deploy
  485  k create -f green-httpd.yaml
  486  k get deploy
  487  k get svc
  488  ls
  489  vi service.yaml
  490  k apply -f service.yaml
  491  k get svc
  492  cat service.yaml
  493  ls
  494  clear
  495  cat service.yaml
  496  k delete svc ext-svc
  497  vi service.yaml
  498  k create -f service.yaml
  499  k get svc
  500  clear
  501  k delete deploy --all
  502  k delete svc --all
  503  clear
  504  k get all
  505  clear
  506  vi blue-nginx-deploy.yaml
  507  vi green-httpd.yaml
  508  vi service.yaml
  509  k create -f green-httpd.yaml
  510  k create -f blue-nginx-deploy.yaml
  511  k create -f service.yaml
  512  clear
  513  k get svc
  514  vi green-httpd.yaml
  515  vi blue-nginx-deploy.yaml
  516  k apply -f green-httpd.yaml
  517  k apply -f blue-nginx-deploy.yaml
  518  clear
  519  curl 172.31.62.63:32446
  520  clear
  521  curl 172.31.62.63:32446
  522  vi blue-nginx-deploy.yaml
  523  k apply -f blue-nginx-deploy.yaml
  524  clear
  525  curl 172.31.62.63:32446
  526  clear
  527  k get deploy
  528  k scale deploy green-httpd --replica 5
  529  k scale deploy green-httpd --replicas 5
  530  curl 172.31.62.63:32446
  531  k scale deploy green-httpd --replicas 10
  532  k get deploy
  533  k scale deploy blue-nginx --replicas 1
  534  k get pods
  535  curl 172.31.62.63:32446

```

ROLLOUT :

```
root@kube-master:~# cat deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 20
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: rk1
  template:
    metadata:
      labels:
        app: rk1
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80



history :

vi deploy.yaml
  592  k create -f deploy.yaml --record=true
  593  k get rs
  594  vi deploy.yaml
  595  k apply -f deploy.yaml --record=true
  596  k get rs
  597  k get deploy
  598  k get rs
  599  vi deploy.yaml
  600  k apply -f deploy.yaml --record=true
  601  k get rs
  602  k rollout history deploy nginx
  603  clear
  604  k rollout history deploy nginx
  605  k describe deploy | grep -i image
  606  k rollout undo deploy nginx --to-revision=2
  607  k get rs
  608  k describe deploy | grep -i image
  609  k rollout undo deploy nginx --to-revision=1
  610  k describe deploy | grep -i image
  611  k rollout undo deploy nginx --to-revision=3
  612  k describe deploy | grep -i image
  613  clear
  614  k get deploy

  615  k get deploy -o yaml
  629  k create deploy dep2 --image httpd --dry-run=client -o yaml > dep2.yaml


```
KUBERNETTES DASHBOARD :

```

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

history :


k get ns
  643  cleaqr
  644  clear
  645  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
  646  k get pods
  647  clear
  648  k get pods -A
  649  clear
  650  k get svc -A
  651  k describe  svc kubernetes-dashboard -n kubernetes-dashboard
  652  k get pods -A
  653  k get pods -A -o wide
  654  clear
  655  k get svc -A
  656  k edit svc kubernetes-dashboard
  657  k edit svc kubernetes-dashboard -n kubernetes-dashboard
  658  k describe  svc kubernetes-dashboard -n kubernetes-dashboard
  659  k get pods -A
  660  k describe pod kubernetes-dashboard-658b66597c-j57lg | grep -i image
  661  k describe pod kubernetes-dashboard-658b66597c-j57lg -n kubernetes-dashboard | grep -i image
  662  clear
  663  k get sa
  664  k get sa -n kubernetes-dashboard
  665  k get roles
  666  k get roles -n kubernetes-dashboard
  667  k describe roles -n kubernetes-dashboard
  668  clear
  669  k get all -n kubernetes-dashboard
  670  k get roles -n kubernetes-dashboard
  671  k describe roles -n kubernetes-dashboard
  672  k describe sa -n kubernetes-dashboard
  673  k describe rolebinding -n kubernetes-dashboard
  674  clear
  675  k get secrets
  676  k get secrets -n kubernetes-dashboard
  677  k describe secret kubernetes-dashboard-token-4245v  -n kubernetes-dashboard
  678  k describe roles -n kubernetes-dashboard
  679  clear
  680  ls
  681  vi admin.yaml
  682  k create -f admin.yaml
  683  k get sa -n kubernetes-dashboard
  684  k describe sa admin-user -n kubernetes-dashboard
  685  k get rolebindings -n kubernetes-dashobard
  686  k get rolebindings -n kubernetes-dashboard
  687  clear
  688  k get roles -A
  689  k get clusterroles -A
  690  k describe clusterrole cluster-admin
  691  clear
  692  vi clusterbinding.yaml
  693  k create -f clusterbinding.yaml
  694  k get sa -n kubernetes-dashboard
  695  k describe sa -n kubernetes-dashboard
  696  clear
  697  kubectl -n kubernetes-dashboard create token admin-user
  698  clear
  699  k get secrets -n kubernetes-dashboard
  700  k describe -n kubernetes-dashboard admin-user-token-2t9q5
  701  k describe secret -n kubernetes-dashboard admin-user-token-2t9q5
  702  clear
  703  k get pods
  704  k create deploy dep2 --image httpd --replicas 10
  705  k get pods
  706  k describe -n kubernetes-dashboard admin-user-token-2t9q5
  707  k describe secret -n kubernetes-dashboard admin-user-token-2t9q5

```


SECRETS :

```

========================

username : cmFtYW5raGFubmEK       ramankhanna
password : cmFtYW5raGFubmExMjMK   -- ramankhanna123







root@kube-master:~# cat secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: opaque
data:
  username: cmFtYW5raGFubmEK
  password: cmFtYW5raGFubmExMjMK








apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      optional: true




==================================================


secret as env :


root@kube-master:~# cat secret2.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: opaque
data:
  username: cmFtYW5raGFubmEK
  password: cmFtYW5raGFubmExMjMK







root@kube-master:~# cat podsecret2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secretpod2
spec:
  containers:
  - name: httpd
    image: httpd
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: password






History :

707  k describe secret -n kubernetes-dashboard admin-user-token-2t9q5
  708  clear
  709  history
  710  clear
  711  k get secrets
  712  clear
  713  echo "ramankhanna" | base64
  714  echo "ramankhanna123" | base64
  715  vi secret.yaml
  716  vi secret.yaml
  717  cat secret.yaml
  718  k create -f secret.yaml
  719  k get secrets
  720  ls
  721  rm -rf secret.yaml
  722  clear
  723  ls
  724  k get secrets
  725  k describe secrets
  726  clear
  727  ls
  728  vi pod.yaml
  729  ls
  730  vi podsecret.yaml
  731  k describe secrets
  732  clear
  733  ls
  734  k create -f podsecret.yaml
  735  k get pods
  736  k delete deploy --all
  737  clear
  738  k get pods
  739  clear
  740  k get pods
  741  k exec -it secretpod -- /bin/bash
  742  clear
  743  k delete secret mysecret
  744  k get secrets
  745  vi secret2.yaml
  746  ls
  747  k create -f secret2.yaml
  748  k get secret
  749  clear
  750  ls
  751  vi podsecret2.yaml
  752  k create -f podsecret2.yaml
  753  k get pods
  754  k exec -it secretpod2 -- /bin/bash
  755  cat secret2.yaml
  756  ls
  757  cat podsecret2.yaml


```

EKS CLUSTER AND ECR :

```
-- i created cluster : control

worker nodes  2


-- created a new env server :

frm thsar servr connect to cont plane eks

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
insalled aws cli



HIstory :


apt install unzip -y
   45  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   46  ls
   47  unzip awscliv2.zip
   48  sudo ./aws/install
   49  aws
   50  clear
   51  aws --version
   52  aws configure
   53  aws
   54  aws s3 ls
   55  clear
   56  aws eks update-kubeconfig --region us-east-1 --name raman-eks
   57  ls -a
   58  cd  .kube
   59  ls
   60  cat config
   61  clear
   62  kubecl get nodes
   63  kubectl get nodes
   64  clear
   65  kubectl -v
   66  kubectl --version
   67  clear
   68  alias k=kubectl
   69  hostnamectl set-hostname eks-controller




 k run app1 --image nginx
   51  k get pods
   52  docker pull ramann123/ibm-repo:ramanimage2
   53  docker images
   54  clear
   55  docker images
   56  aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 685421549691.dkr.ecr.us-east-1.amazonaws.com
   57  docker tag ramann123/ibm-repo:ramanimage2 685421549691.dkr.ecr.us-east-1.amazonaws.com/raman-repo:ramanimage
   58  docker images
   59  docker push 685421549691.dkr.ecr.us-east-1.amazonaws.com/raman-repo:ramanimage
   60  clear
   61  k run app2 --image public.ecr.aws/docker/library/redis:alpine3.18

```




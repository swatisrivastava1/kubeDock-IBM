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

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


```

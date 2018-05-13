# K8s setup
**Ubuntu 16.04**

Notes on setting up a custom `kubernetes` cluster within AWS


## Installing Docker
First install `apt-transport-https` - a package that allows using https as well as http in apt repository sources.    
1. `sudo apt-get update && sudo apt-get install -y apt-transport-https`    
Next install `docker`    
2. `sudo apt install docker.io`     
Now start and enable the Docker service     
3. `sudo systemctl start docker && sudo systemctl enable docker`      

*Viola! Now we can install Kubernetes.*

## Installing Kubernetes
Download and add the key    
1. `sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add`    
 
2. Create the file `/etc/apt/sources.list.d/kubernetes.list` with the following contents
```
deb http://apt.kubernetes.io/ kubernetes-xenial main 
```    
3. `sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni`    

*Viola! Now you can setup your cluster.*

## Master Installation
+ Init *Kube Master* with a flag for `Flannel` later: `sudo kubeadm init --pod-network-cidr=10.244.0.0/16`
+ Save the join command for later. (If you forget to, you can always run: `sudo kubeadm token create --print-join-command`)
+ `mkdir -p $HOME/.kube`
+ `sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
+ `sudo chown $(id -u):$(id -g) $HOME/.kube/config`

*Viola! Now you can setup Flannel.*

## Flannel Installation
+ Must pass bridged IPv4 traffic to iptables’ chains. Ensure that `/proc/sys/net/bridge/bridge-nf-call-iptables` is `1`. If not: `sysctl net.bridge.bridge-nf-call-iptables=1`
+ `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml`
+ Confirm that it is working by checking that the kube dns pod is up and running: `kubectl get pods --all-namespaces`

## Connecting Nodes
Install `Docker` and `Kubernetes`, then run the previously saved join command. 

If you need to run `kubectl` on a worker node for some reason, copy `~/.kube/config` from the master onto the workers filesystem. Then you can run `kubectl --kubeconfig=~/.kube/config get pods`.

## Nginx Controller
Follow this gist: `https://gist.github.com/azban/80e816e6873574cf941346bb5fc30ca4`

## Dashboard Installation
**This needs an update, it does NOT work**
Follow this gist:
+ `https://gist.github.com/azban/9e6ed46048ff29f83a0f19fd36fe9a93`   
+ You might need to restart the `heapster` and `dashboard` pods in the `kube-system` namespace to get things working. 

Now setup an ingress:   
+ `kubectl apply -f ./k8s/dashboard/ingress.yaml`

## ROS Setup
+ Clone our repo: `git clone https://github.com/Automa-Cognoscenti/ops.git`
+ `kubectl apply -f k8s/ros`
+ Now you can check that the `listener` heard the `talker`: `kubectl logs \`kubectl get pods | grep listener | awk ' { print $1 } '\``


## References
* https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/
* https://www.techrepublic.com/article/how-to-quickly-install-kubernetes-on-ubuntu/
* https://askubuntu.com/questions/833114/how-to-set-up-ethernet-connection-16-04-server

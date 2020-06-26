Assuming WSL2 is already enabled and using Ubuntu 20.04 LTS.

1. [Install `docker`](https://docs.docker.com/engine/install/ubuntu/):
   ```
   sudo apt-get update

   sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg-agent \
     software-properties-common

   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    
   sudo apt-key fingerprint 0EBFCD88

   sudo add-apt-repository \
     "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) \
     stable"

   sudo apt-get update

   sudo apt-get install docker-ce docker-ce-cli containerd.io

   # if prompted about grub during install, select all drives and then select yes when the message pops up again

   sudo docker run hello-world
   ```
   If you'd rather not have to use `sudo` then also do:
   ```
   sudo usermod -aG docker $USER
   ```
   Then close and reopen WSL2, restart the Lxss service, or sign in and out of Windows... which ever works. 
   
   Of course, every time you boot up Windows you need to start `docker` in WSL2 with:
   ```
   sudo service docker start
   ```

2. [Install `kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/):
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

    chmod +x ./kubectl

    sudo mv ./kubectl /usr/local/bin/kubectl
    ```

3. [Install `minikube`](https://kubernetes.io/docs/tasks/tools/install-minikube/):
     ```
    curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
      && chmod +x minikube

    sudo mkdir -p /usr/local/bin/

    sudo install minikube /usr/local/bin/
    ```
    Go ahead and create a VM while we are here. 
    ```
    minikube start --vm-driver=docker --kubernetes-version=1.15.10
    ```
    That is just the specific version of Kubernetes I use. 

    `--vm-driver=none` doesn't work. `--vm-driver=podman` works about as well as `--vm-driver=docker` in that you can start a cluster, but can't connect to it from the outside, which also makes it unusable.

    The `minikube` VM should spin up in about a minute. 
    
    Run the following to get the IP that we want to connect to from browser:
    ```
    minikube ip
    ```
    Beacuse we are using `--vm-driver=docker`, it will likely be `172.17.0.3`.

    If you run into an error, either `docker` isn't running or you are hitting the notorious "`cgroups`" error which can be fixed with... `alias` it because you'll be running it often:

    ```
    sudo mkdir /sys/fs/cgroup/systemd

    sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
    ```

4. [Install `skaffold`](https://skaffold.dev/docs/install/):
    ```
    curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64

    sudo install skaffold /usr/local/bin/
    ```

5. [Install `ingress-nginx`](https://kubernetes.github.io/ingress-nginx/deploy/)
    ```
    # this command may no longer be necessary, but I still run it without issues in macOS and Linux
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
    
    # this command may no longer be necessary, but I still run it without issues in macOS and Linux
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/cloud-generic.yaml
    
    minikube addons enable ingress
    ```

6. 
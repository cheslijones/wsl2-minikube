# Setup

Assuming WSL2 is already enabled and using WSL2 Ubuntu 20.04 LTS is already installed.

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

5. [Install / enable `ingress-nginx`](https://kubernetes.github.io/ingress-nginx/deploy/)
    ```    
    minikube addons enable ingress
    ```

6. `git clone https://github.com/eox-dev/minikube-wsl2.git`

7. `cd minikube-wsl2`

8. `skaffold dev`

This should spin up the cluster. If not:

- Make sure `docker` is running (`sudo service docker start`)
- If it is running, then you are likely getting the "`cgroups` error:
  ```
  sudo mkdir /sys/fs/cgroup/systemd

  sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
  ```

# Expectiations vs Reality

With the cluster happily runing, and with your `minikube ip` in hand, go to the IP address in browser. Mostly likely it is `172.17.0.3`. 

The browser will spin and eventually say it can't be located. Only happens with `--vm-driver=docker`... well `--vm-driver=podman` also. 

One proposed solution is to use `--port-forward`. If I run `skaffold dev --port-forward`, you can navigate to the microservices running in you cluser with `localhost:3000` (`3000` being the port `npm start` is running on). But this is highly undesirable as it will bypass cluster routing issues, which defeats the purpose of a local Kubernetes development environment. 

For example, when I was setting up a Django API and needed to prefix `/api` to the route such that Django Admin portal continued to work. If I had used `--port-forward`, I wouldn't have come across an issue where Django admin wasn't working because navigating to `/api/admin` would automatically reroute to `/admin` which wasn't a route in the the `ingress.yaml` and shouldnt' be. By running a local Kubernetes dev cluster, without `--port-forward`, I figured out I needed to add `FORCE_SCRIPT_NAME = '/api/'` to the `settings.py` and the problem was solved.

Another proposed solution is to use `minikube tunnel`. I've never been able to get this to work. I get dozens of:

```
Status:
        machine: minikube
        pid: 4065
        route: 10.96.0.0/12 -> 172.17.0.3
        minikube: Running
        services: [ingress-nginx]
    errors: 
                minikube: no errors
                router: no errors
                loadbalancer emulator: no errors
```
A new entry is added any time I navigate to `10.96.0.x` or `172.17.0.3`. Ultimately, it just results in the same. Browsers says "Hmmmm... can't reach this page."

Also have tried `:3000` at the end of these IP addresses. Doesn't work and shouldn't because the ingress should be routing traffic on `80` or `443`.

# Final Notes

This setup is working fine in:

- macOS with `--vm-driver=hyperkit`
- Linux (Pop!_OS specifically) with `--vm-driver=kvm2`... I do run into the same issues with `--vm-driver=docker` in Pop!_OS, but no reason at all to use it there. Forced to use it in WSL2, because it doesn't support nested virtualization yet. Rebuilding the WSL2 kernel to enable nested virtualization works for some, but not others. I've never been able to get it working, personally.
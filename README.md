# Purpose
This repo was originally intended to show the numerous issues one was faced with when trying to use WSL2 with `minikube`.

Now, after several years and countless hours of testing to get issues worked out, it finally looks like WSL/WSL2 is usable for development (for my use case). Much appreciation to the developers of WSL2 and `minikube`. I've actually deleted my Linux partition and started using WSL2 fulltime.

My use case is the following stack:
- `docker` running in WSL2 and not Docker Desktop for Windows
- `kubectl`
- `minikube`
- `skaffold`
- `mkcert` for local development TLS certificates

I'm repurposing this repo to demonstrate how to get this stack up and running in WSL2.

For the `mkcert` implementation, please see [this repo](https://github.com/cheslijones/tls-minikube).

# Setup

Assuming WSL2 is already enabled and using WSL2 Ubuntu 20.04 LTS is already installed.

Refer to the documentation below for the latest installation processes.

1. `git clone https://github.com/cheslijones/wsl2-minikube.git`

2. `cd wsl2-minikube`

3. [Install `docker`](https://docs.docker.com/engine/install/ubuntu/) directly into WSL2:

   If you'd rather not have to use `sudo` then also do:
   ```
   sudo usermod -aG docker $USER
   ```
   Then close and reopen WSL2, restart the Lxss service, or sign in and out of Windows... which ever works. 
   
   Of course, every time you boot up Windows you need to start `docker` in WSL2 with:
   ```
   sudo service docker start
   ```

4. [Install `kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/)


5. [Install `minikube`](https://kubernetes.io/docs/tasks/tools/install-minikube/):
    
    Go ahead and create a cluster while we are here. 
    ```
    minikube start --vm-driver=docker
    ```
    It should spin up in about a minute. 

6. [Install `skaffold`](https://skaffold.dev/docs/install/)

7. [Install / enable `ingress-nginx`](https://kubernetes.github.io/ingress-nginx/deploy/):

    It is a bit counter-intuitive, but using the Docker for Mac instructions works not only with Docker Desktop for Windows and macOS, but also WSL2:
    ```    
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.45.0/deploy/static/provider/cloud/deploy.yaml
    ```

8. Run `skaffold dev`.
9. In another terminal, run `minikube tunnel` which should prompt you for `sudo` credentials:

    ```
    $ minikube tunnel
    ❗  The service ingress-nginx-controller requires privileged ports to be exposed: [80 443]
    🔑  sudo permission will be asked for it.
    🏃  Starting tunnel for service ingress-nginx-controller.
    [sudo] password for cheslijones: 
    ```

10. Open your preferred browser and navigate to `localhost`. 

    You should see:

    [<img src="https://assets.digitalocean.com/articles/66983/React_template_project.png">](https://assets.digitalocean.com/articles/66983/React_template_project.png)

    If not:

    - Make sure `docker` is running (`sudo service docker start`).
    - Make sure your `minikube` cluster is running with `minikube start`.


# Questions, Issues and Feedback
Please create an issue if you have any questions, issues running the repo, or have feedback on how I can improve this repo or correct something that is wrong. I'm always looking for ways to improve.

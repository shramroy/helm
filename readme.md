# Ixia-C Deployment Using Helm Chart

Step1: Kubernetes cluster creation

1. Deploy a clean instance of Ubuntu Server VM with following prerequisites:
    - Recommended 4 CPU cores (Controller with one TE)
    - Recommended 10GB RAM

2. Login to newly deployed server and get helper scripts.

    ```sh
        git clone http://gitlab.it.keysight.com/athena/scripts.git
        cd scripts/kne && chmod +x ./kne.sh
    ```
      

3. Install prerequisites and setup kubeadm cluster.
    - comment out meshnet, go and kne part in kne.sh 

    ```sh
        ./kne.sh setup
    ```
    - ensure all pods are up and running and kubectl will be installed automatically as part of kubeadm setup
    ```sh
        kubectl get pods -A
    ```


Step2: Install helm 

```sh
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    chmod 700 get_helm.sh
    ./get_helm.sh
```

Step3: Install helmfile
- References:
    - https://github.com/roboll/helmfile
    - https://www.codegrepper.com/code-examples/shell/helmfile+install+ubuntu

    ```sh
        wget -O helmfile_linux_amd64 https://github.com/roboll/helmfile/releases/download/v0.135.0/helmfile_linux_amd64
        chmod +x helmfile_linux_amd64
        mv helmfile_linux_amd64 ~/.local/bin/helmfile
    ```

    OR

    Install brew package in ubuntu

    ```sh 
        sudo apt update
        sudo apt-get install build-essential
        sudo apt install git -y
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
        eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
        brew doctor
        brew install gcc
    ```

    Install helmfile

    ```sh
        brew install helmfile
    ```

Step4: Interface Creation

```sh
    sudo ip link add veth1 type veth peer name veth2
    sudo ip link set veth1 up
    sudo ip link set veth2 up
```

Step5: Deployment using helmfile
- add helm chart repo

```sh
    helm repo add <local_repo_name> <location_of_helm_chart>
```
Ex- 

```sh
    helm repo add helm https://shramroy.github.io/helm
```
- check all the local helm chart repo list 

```sh
    helm repo list -a
```
- search helm charts 

```sh 
    helm search repo <local_repo_name>
```
- check helm envioronment variables

```sh 
   helm env 
```
- check the releases of helm chart based on the namespace 

```sh 
   helm list -a 
```
- check all contents(in chart.yaml & values.yaml) for a specific helmchart

```sh 
    helm show all helm/my-chart
```
- check contents in Chart.yaml for a specific helmchart

```sh 
    helm show chart helm/my-chart
```
- check contents in values.yaml for a specific helmchart

```sh 
    helm show values helm/my-chart
```
- check the details about the release of helm chart 

```sh 
    helm status <release_name>
```

- Get samples of helmfile (b2b, dise)

- Edit the 'chart' in helmfile.yaml based on the local location of the helm chart for that topology
  Ex- 
  ```sh
  releases:
  - chart: helm/ixia-c
  ```

- Modify helmfile based on requirements like interace & port details

- Deployment/Install:

    - Go to specific topology directory 
    ```sh
        cd topologies/b2b
    ```

    - Run:
    ```sh
        helmfile sync
    ```


- Delete/Uninstall:

    - Update helmfile.yaml with `installed: false` for application(s)

    - Run: 
    ```sh
        helmfile sync
    ```

- Use exposed services on given Nodeports:
    - Execute the following command to know the nodeports of exposed serives
    - Run: 
    ```sh
        kubectl get svc -n <namespace_name>
    ```
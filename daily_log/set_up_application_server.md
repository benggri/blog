# Set up application server on my mini pc

- Actually I bought 3 mini PCs.
1. For development in Windows.
1. Database server running Ubuntu Server 24.04, set up to host PostgreSQL, MongoDB, Redis, and Kafka (although Kafka isn't a database... but this PC is named "Database Server").
1. Application server running Ubuntu Server 24.04, designed to host applications on Kubernetes.

## Application server set up log

1. Install ubuntu server 24.04
    - I didn’t install OpenSSH or Docker during the Ubuntu Server installation.
1. Docker install
    ```bash
    # Remove the existing Docker installation (if it is already installed).
    sudo apt-get remove docker docker-engine docker.io containerd runc
    snap remove docker

    # Install the required packages.
    sudo apt-get update

    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release

    # Install Docker(including keyings)
    sudo mkdir -m 0755 -p /etc/apt/keyrings

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    sudo apt-get update
    
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    # Check the Docker installation results
    docker --version
    
    docker images

    # If you encounter a "Permission denied" error, please run the command below.
    sudo chmod 777 /var/run/docker.sock
    sudo usermod -aG docker $USER
    newgrp docker
    ```
1. Setting up a static IP
    - (I just thought, shouldn't the Database Server have a static IP?)
    1. I have modified the **.yaml** file in the **/etc/netplan** folder.
        ```bash
        cd /etc/netplan
        sudo cp <file_name>.yaml <file_name>.yaml # I make a copy for backup before restoring.
        ```
        - file
        ```yaml
        network:
            version: 2
            renderer: NetworkManager
            ethernets:
                enp1s0: <= might vary depending on the device
                    dhcp4: no
                    addresses:
                        - <static_ip_to_be_used>/24
                    routes:
                        - to: default
                          via: <gateway_ip>
                    nameservers:
                        addresses:
                            - 8.8.8.8
                            - 8.8.4.4
        ```
    1. Finding the gateway IP
        1. Naturally, I thought I could find it using the **ifconfig** command.
        1. I wondered if I could find it with the **route** command, but that wasn't the case.
        1. It turns out I could find it using the **ip route** command.
        - (Actually, I executed many more commands besides these...)
    1. After modifying the file, I apply the changes.
        ```bash
        sudo netplan apply
        ```
        - Then, I verify if the changes have been applied correctly.
        ```bash
        ping -c 4 8.8.8.8
        nslookup google.com
        ```
    - Useful tools related to IP and networking?
        1. Install nmcli
            ```bash
            sudo apt-get install network-manager
            ```
        3. How to use it
            ```bash
            nmcli d
            # The command results are displayed in a neat format (with colors as well).
            ```
            DEVICE | TYPE | STATE | CONNECTION | Color 
            --- | --- | --- | --- | ---
            enp1s0 | ethernet | connected | netpaln-enp1s0 | green
            docker0 | bridge | connected (externally) | docker0 | gray
            another2 | wifi | disconnected | -- | red
1. Install **openssh-server**
    ```bash
    sudo apt-get install openssh-server
    ```
    ```bash
    sudo service ssh status
    ```

1. Install **kubernetes**
    1. References
        - [Installing kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)
        - [Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
    - **kubeadm**: the command to bootstrap the cluster.
    - **kubelet**: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
    - **kubectl**: the command line util to talk to your cluster.
    ```bash
    sudo apt-get update
    # apt-transport-https may be a dummy package; if so, you can skip that package
    sudo apt-get install -y apt-transport-https

    # If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
    # sudo mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    # This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl

    sudo systemctl enable --now kubelet
    ```

1. Run **kubernetes**
    ```bash
    kubeadmin init
    ```
    - An error related to containerd has occurred.
        - kubeadm init failed to create new cri runtime service~~
        ```bash
        sudo rm /etc/containerd/config.toml
        sudo systemctl restart containerd
        sudo kubeadm init
        ```







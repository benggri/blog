# Set up database server on my mini pc

- Actually I bought 3 mini PCs.
1. For development in Windows.
1. Database server running Ubuntu Server 24.04, set up to host PostgreSQL, MongoDB, Redis, and Kafka (although Kafka isn't a database... but this PC is named "Database Server").
1. Application server running Ubuntu Server 24.04, designed to host applications on Kubernetes.

## Database server set up log

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
        sudo cp <file_name>.yaml <file_name>.yaml
        ```
        ```yaml
        network:
            version: 2
            renderer: NerworkManager
            ethernets:
                enp1s0:
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

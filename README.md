# EDC-MVD-IONOS-VM
How to deploy EDC Minimum Viable Data Space on IONOS cloud Virtual Machines 
# Minimum Viable Dataspace IONOS Virtual Machine


This GIT repository allows anyone, through some requirements, to deploy quickly an up and running virtual machine with the [Eclipse Minimum Viable Dataspace](https://github.com/eclipse-dataspaceconnector/MinimumViableDataspace) on the IONOS Cloud.


## Documentation

Some pointers:
- [Eclipse Minimum Viable Dataspace](https://github.com/eclipse-dataspaceconnector/MinimumViableDataspace);
- [Local](https://docs.google.com/presentation/d/1LHTHXpZG0ZC_eWgKv4B0gb71XCbdk3WLjsL4MDcV6NQ/edit?usp=sharing) installation of the MVD, step by step;
- Installation of the MVD using the [IONOS DataCenter Designer](https://docs.google.com/presentation/d/13oBbd2cuu_KbCwk41NAsm4yK_HofGdCEHiQckCLOkC8/edit?usp=sharing);
- [Go Language](https://go.dev);
- Go Language [Install](https://medium.com/@benzbraunstein/how-to-install-and-setup-golang-development-under-wsl-2-4b8ca7720374)
- [Terraform](https://www.terraform.io);
- IONOS Cloud Provider [Documentation](https://registry.terraform.io/providers/ionos-cloud/ionoscloud/latest/docs) - Terraform;
- IONOS Cloud Provider [Documentation](https://github.com/ionos-cloud/terraform-provider-ionoscloud/) - IONOS;

## Requirements

You will need the following:
- IONOS Cloud account;
- Go Language;
- Terraform installed locally;
- GIT;

## Requirements for Installation and Configuration

If you have the requirements already fulfilled, skip this part and go to `Executing the terraform scripts` section.

> This guide was made using the WSL2 for Windows (Ubuntu 20.04.5 LTS).

### Go Language

Follow these steps:

```bash
wget https://go.dev/dl/go1.19.3.linux-amd64.tar.gz
sudo tar xvzf go1.19.3.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$PATH
```
Add the last 3 lines to your .bashrc file.

Check if it is running with 
```bash
go version
output: go version go1.13.8 linux/amd64
```

### Terraform

Follow these steps:

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

Check if it is running with 
```bash
terraform version
output: 
Terraform v1.3.5
on linux_amd64
+ provider registry.terraform.io/ionos-cloud/ionoscloud v6.3.3
```

### Terraform IONOS Provider

Follow these steps:

```bash
mkdir -p $GOPATH/src/github.com/ionos-cloud; cd $GOPATH/src/github.com/ionos-cloud
git clone https://github.com/ionos-cloud/terraform-provider-ionoscloud.git
cd $GOPATH/src/github.com/ionos-cloud/terraform-provider-ionoscloud
make build
```

## Create Virtual Machine with MVD

### Cloning the repository

You will have to clone this repository:
```bash
git clone https://github.com/ReadyPJC/IONOS_MVD_VM.git
```

### SSH Key Pair

Before we continue, you will have to do some changes regarding the SSH key pair required for the configuration of the VM and for the SSH access. It is needed in two phases:
1. public key, that will be installed in the server in order to you to have a secure connection;
2. private key, that will be used by the terraform itself to do some configuration on the server;

If you don't have one ssh key pair, you can create it using for example the [PuTTYgen](https://www.puttygen.com/) app.

With your key pair, do the following steps:
- copy the public key to the `ssh/ssh_public.pk` file or to another location. If you do that, don't forget to change the path of the `ssh_publickey_path` variable.
- copy the private key (SSH format, not the PuTTY one) to the `ssh/id_rsa` file or to another location. If you do that, don't forget to change the path of the `ssh_privatekey_path` variable.

Now, we are ready to create the server and deploy the MVD!

### Executing the terraform scripts

First we need to provide some authentication data allowing the IONOS Cloud Provider to connect. According to this [site](https://registry.terraform.io/providers/ionos-cloud/ionoscloud/latest/docs), there is several ways to do it. In this guide, we will export the IONOS account (username and password) to environment variables:
```bash
export IONOS_USERNAME="username" >>>> put your IONOS Cloud account username
export IONOS_PASSWORD="password" >>>> put your IONOS Cloud account password
```
Then execute the following:
```bash
terraform init
terraform plan -var="location=de/fkb" -var="project_name=EDV_MVD" -var="environment=dev"
terraform apply -var="image_password=SUPERSECRETPASSWORD"
```
Notes:
- terraform init, will initialize the IONOS Cloud Provider;
- terraform plan, will check and prepare the required infrastructure. You can change the values of the variables;
- terraform apply, will create the server and the necessary infrastructure, configure it and start the MVD. The variable `image_password` will be your root password;
- you will have to wait a few minutes before accessing the MVD on your SSH client app or browser;

At the end of the execution, you will see on your terminal, the IP address of your server, for instance: mvd_server_ip = "85.215.161.38"

## Accessing the server

Now that the server is up and running, you can access the server with an SSH client application and the MVD with your favorite browser. For ssh, don't forget to point to the IP server and to select your private key.

If you want, you can access the server to check if everything is up and running. In order to do that:
- access the server with ssh with the user `root`;
- go to the directory `/root/edc/MinimumViableDataspace`;
- check (for instance with `tail -f nohup.out`) the file `nohup.out` to see if the deployment of the MVD is completed;
- if the ouput is similar to the following, wait a few more minutes:
```bash
ec07c04a8d4c Downloading [>                                                  ]  31.79kB/2.239MB
396c31837116 Downloading [>                                                  ]  31.79kB/2.798MB
9e7b0c9574dd Downloading [>                                                  ]  374.5kB/36.01MB
ec07c04a8d4c Downloading [===============================>                   ]  1.407MB/2.239MB
396c31837116 Downloading [=============================>                     ]  1.669MB/2.798MB
```
- if the output is similar to the following, you can access the MVD with your browser:
```bash
registration-service    | 2022-11-23 16:33:10 FINE : Valid JWT
registration-service    | 2022-11-23 16:33:10 INFO : List all participants of the dataspace.
company1-datadashboard  | 127.0.0.1 - - [23/Nov/2022:16:33:10 +0000] "GET / HTTP/1.1" 200 1511 "-" "curl/7.83.1"
company1                | 2022-11-23 16:33:10 WARNING : No WorkItems found, aborting execution
company1                | 2022-11-23 16:33:10 INFO : ExecutionManager: Run post-execution task
company2-datadashboard  | 127.0.0.1 - - [23/Nov/2022:16:33:10 +0000] "GET / HTTP/1.1" 200 1511 "-" "curl/7.83.1"
company2                | 2022-11-23 16:33:10 INFO : ExecutionManager: Run pre-execution task
company2                | 2022-11-23 16:33:10 INFO : ExecutionManager: Run execution
company3                | 2022-11-23 16:33:10 INFO : ExecutionManager: Run pre-execution task
company3                | 2022-11-23 16:33:10 INFO : ExecutionManager: Run execution
did-server              | 172.18.0.4 - - [23/Nov/2022:16:33:10 +0000] "GET /company2/did.json HTTP/1.1" 200 924 "-" "okhttp/4.10.0" "-"
```

## Start/Stop the MVD

If you want to stop/start the MVD, access the MVD server with your SSH client, go to the `/root/edc/MinimumViableDataspace` directory, and execute the following:
- stop: docker-compose -f system-tests/docker-compose.yml stop
- start: docker-compose --profile ui -f system-tests/docker-compose.yml up --build

## Removing the server

To delete all the infrastructure from the IONOS cloud, execute the following:
```bash
terraform destroy
```

Notes:
- you could have to repeat the command because of the dependencies between the components of the infrastructure;
- you can also delete the server using the [DataCenter Designer](https://dcd.ionos.com) from IONOS; 

## Using the MVD

To use the MVD one could do the following:
1. Create a text file (with any content) with the name `text-document.txt`. This file is going to be uploaded to the azure storage of the `company1`;
2. Upload the file:
    - Using an app: With the MVD running, you can explore this prototype using this [document](https://docs.google.com/presentation/d/13oBbd2cuu_KbCwk41NAsm4yK_HofGdCEHiQckCLOkC8/edit?usp=sharing), starting slide 13.
    - Using a command line:
        - First, you must install the Azure CLI on your machine ([Linux](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux) or [Windows](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows));
        - Uploading the file (linux):
        ```bash
            conn_str="DefaultEndpointsProtocol=http;AccountName=company1assets;AccountKey=key1;BlobEndpoint=http://<SERVER_IP>:10000/company1assets;"
            az storage container create --name src-container --connection-string $conn_str
            az storage blob upload -f ./deployment/terraform/participant/sample-data/text-document.txt --container-name src-container --name text-document.txt --connection-string $conn_str
        ```
3. Using your browser you can access the sites of the 3 companies: `company1 - http://<SERVER_IP>:7080`, `company2 - http://<SERVER_IP>:7081` and `company3 - http://<SERVER_IP>:7082`;
4. Going to the company1's site and choosing the option `Assets`, you will see the `text-document` file;
5. Going to the compnay2's site and choosing the option `Catalog Browser`, you will have access to the files of the 3 companies. Let’s say the company2 wants to deal with the company1 the `test-document` file. We just have to click `Negotiate` and a message will appear: `Contract Negotiation complete! Show me!`;
6. We now have a new contract created between Company1 and Company2. Using the option `Contracts` from the Company2's site we can begin the transfer of the file clicking `Transfer` and selecting `AzureStorage`. We can see the result of the transfer using the `Transfer History` option;
7. To verify that there is a new file into the storage of the Company2, you can use the GUI App or the Azure CLI;


For more details, take a look at those documents: [Local](https://docs.google.com/presentation/d/1LHTHXpZG0ZC_eWgKv4B0gb71XCbdk3WLjsL4MDcV6NQ/edit?usp=sharing) deployment and using the [IONOS DataCenter Designer](https://docs.google.com/presentation/d/13oBbd2cuu_KbCwk41NAsm4yK_HofGdCEHiQckCLOkC8/edit?usp=sharing);


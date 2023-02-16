# EDC-MVD-IONOS-VM
How to deploy EDC Minimum Viable Data Space on IONOS cloud Virtual Machines 


## Based on the following

- [https://github.com/eclipse-dataspaceconnector/DataSpaceConnector](https://github.com/eclipse-dataspaceconnector/DataSpaceConnector);
- [https://github.com/eclipse-dataspaceconnector/MinimumViableDataspace](https://github.com/eclipse-dataspaceconnector/MinimumViableDataspace);
- [https://github.com/eclipse-dataspaceconnector/MinimumViableDataspace/blob/main/system-tests/README.md](https://github.com/eclipse-dataspaceconnector/MinimumViableDataspace/blob/main/system-tests/README.md);

## Requirements

You will need the following:
- Java Development Kit (JDK) 11 or higher;
- Javadoc;
- Docker;
- GIT;
- Linux shell or PowerShell;

## Deployment

### Cloning the projects

```bash
mkdir /EDC
cd /EDC
git clone https://github.com/eclipse-dataspaceconnector/MinimumViableDataspace
git clone https://github.com/eclipse-dataspaceconnector/DataDashboard
```

### Building projects

```bash
cd /EDC/MinimumViableDataspace
./gradlew build -x test
./gradlew -DuseFsVault="true" :launchers:connector:shadowJar
./gradlew -DuseFsVault="true" :launchers:registrationService:shadowJar
```

### Making the DataDashboard directory PATH available system-wide

```bash
sudo nano /etc/environment
```
and add the following line to the end:
```bash
MVD_UI_PATH="/EDC/DataDashboard"
```
Exit nano, by Ctrl+x and y to save the file.

Make it available without rebooting:
```bash
source /etc/environment
```

### Run the MVD

```bash
docker-compose --profile ui -f /EDC/MinimumViableDataspace/system-tests/docker-compose.yml up --build
```
You should have an output similar to the following:

![MVD_RUNNING](images/mvd_running.png)

The shell will constantly output informations about this MVD Ecosystem.
Notice the colors of the companies. We will get back to it.

## Usage

### Azureit

The files of the 3 Companies are being kept in local Azure storage, called [Azurite](https://github.com/Azure/Azurite).
In order to access the data we can use an application or specific command lines. We are going to use an application called Microsoft Azure Storage Explorer - [MASE](https://azure.microsoft.com/features/storage-explorer/)

- Select “Local storage emulator” from the “Select Resource” option

![MASE](images/mase1.png)

- Configure each company with the following data: company1assets/key1, company2assets/key2 and company3assets/key3

![MASE](images/mase2.png)

You should have something similar with the following:

![MASE](images/mase3.png)

- Create a new blob container for Company1 called “src-container”

- Create a file “test-document.txt” and upload it to “src-container”

If you prefer, you can use the Azure Cli to execute the previous commands. Consider the following:

- You must install the Azure CLI on your machine ([Linux](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux) or [Windows](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows));
- Upload the file (linux):
```bash
conn_str="DefaultEndpointsProtocol=http;AccountName=company1assets;AccountKey=key1;BlobEndpoint=http://127.0.0.1:10000/company1assets;"
az storage container create --name src-container --connection-string $conn_str
az storage blob upload -f ./deployment/terraform/participant/sample-data/text-document.txt --container-name src-container --name text-document.txt --connection-string $conn_str
```

### Websites

- Every Company has its own website accessible through: http://localhost:7080 for company 1 (company 2 - 7081 and compaany 3 - 7082). Each company site has its own color ("Getting Started"). They are the same as the console output.
- Let’s start the process of transferring a file between company1 and company2. Through the Company2 site, option “Catalog Browser”, we can see all the files of the 3 companies. Let’s say the company2 wants to deal with the company1 the “test-document” file. We just have to click `Negotiate` and a message will appear. 

![COMPANY2](images/website1.png)

- We now have a new contract created between Company1 and Company2. Using the option “Contracts” from the Company2 site we can begin the transfer of the file, clicking `Transfer` and selecting `AzureStorage` (Azurite). We can see the result of the transfer using the `Transfer History` option.

![COMPANY2](images/website2.png)

- We can see, using the MASE, that there is a new container for company2 with two files: .complete and text-document.txt. Let’s open this one to check if it’s what we created earlier.

![COMPANY2](images/website3.png)

## Stop

To stop the MVD, just do a CTRL-C.


For more details, check the `Based on the following` section.


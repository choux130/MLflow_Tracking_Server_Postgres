# MLflow Tracking Server with Postgres

### Key Words
<code>MLflow</code>, <code>Docker</code>, <code>SSH</code>, <code>Azure Web App</code>, <code>Azure Registry</code>, <code>Auzre Blob Storage</code>, <code>Postgres</code>, <code>docker-compose</code>

### Introduction 
This project is based on [choux130/MLflow_Tracking_Server](https://github.com/choux130/MLflow_Tracking_Server/edit/master/README.md) but instead of storing all output locally in the container, we store in the Postgres database container and then back up in the docker volume. So, if for some reasons we need to restart our mlflow container, we won't lose the data. 


### Steps
1. Create a "Storage Account" in Azure. The place where model aritacts are going to save. [Create a BlockBlobStorage account](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-create-account-block-blob?tabs=azure-portal)
2. Fill in all the credential in <code>Dockerfile</code> as environment variables.   
3. Test the deployment locally using 
    ```bash
    docker-compose build
    docker-compose up
    ```   
    Open [localhost:5000](localhost:5000) to make sure the MLflow ui show up correctly. And test if the running container can be connected through SSH. If the error message shown up, <code>Unable to negotiate with ::1 port 2222: no matching cipher found. Their offer: aes128-cbc,3des-cbc,aes256-cbc</code>, then take a look at this post. [ssh error: unable to negotiate with IP: no matching cipher found](https://ma.ttias.be/ssh-error-unable-negotiate-ip-no-matching-cipher-found/)
    ```bash
    ssh root@localhost -p 2222
    ```
    Try to connect to the Postgres <code>mlflow</code> database and query the tables.
    ```
    docker exec -t -i postgres_db /bin/bash
    ```
    ```
    psql mlflow admin # connec to the mlflow database with username admin 
    \dt # list all the tables
    SELECT * FROM metrics;
    ```
4. Create a "Azure Registry" in Azure if it has not been created. [Quickstart: Create a private container registry using the Azure portal](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal)
5. Push image to the created Azure Registry. [Push your first image to a private Docker container registry using the Docker CLI](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli) [Push to private container registry with docker-compose
](https://stackoverflow.com/questions/44052999/docker-compose-push-image-to-aws-ecr)
5. Create "Web App" in Azure using the images pushed to the Azure Registry. And then, upload the <code>docker-compose.yml</code> file. (remove the build part)
[Deploying Containerised Apps to Azure Web App for Containers](https://chrissainty.com/containerising-blazor-applications-with-docker-deploying-containerised-apps-to-azure-web-app-for-containers/)

### Quick Test
1. Set environment variables, <code>MLFLOW_TRACKING_URI</code> and <code>AZURE_STORAGE_CONNECTION_STRING</code>. 

    Linux and MacOS
    ```bash
    export MLFLOW_TRACKING_URI=https://<web-app-name>.azurewebsites.net
    export AZURE_STORAGE_CONNECTION_STRING='<connection-string>'
    ```
    Windows
    ```bash
    set MLFLOW_TRACKING_URI=https://<web-app-name>.azurewebsites.net
    set AZURE_STORAGE_CONNECTION_STRING='<connection-string>'
    ```
2. Have python package installed.
   ```bash
   pip install mlflow==1.7.2
   pip install azure-storage-blob==2.1.0
   ```
3. Open Python and make sure the environment variables have been correctly passed. 
    ```python
    import os 
    print(os.getenv('MLFLOW_TRACKING_URI'))
	print(os.getenv('AZURE_STORAGE_CONNECTION_STRING'))
    ```
4. Try to log parameters, metric and artifacts to the tracking server.
    ```python
    import mlflow
    
    mlflow.start_run()
    mlflow.log_param("param1", 500)
    mlflow.log_metric("foo", 100)
 
    with open("output.txt", "w") as f: f.write("Hello world!")
    mlflow.log_artifact("output.txt")   
    mlflow.end_run()
    ```
5. Go to the web app (<code>https://\<web-app-name\>.azurewebsites.net</code>) and see if the run has been logged and go to blob storage to make sure the artifacts have been saved in the container.

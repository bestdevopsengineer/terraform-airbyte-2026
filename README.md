# terraform-airbyte-2026

# Airbyte platform
  open source data integration
   It helps you consolidate data from hundreds of sources into your:
     -data warehouses, 
     -data lakes, and 
     -databases.
   it helps you move data from those locations into the operational tools where work happens, like :
     -CRMs, 
     -marketing platforms, and 
     -support systems.
## Airbyte plans
Airbyte is available as a 
  -self-managed Enterprise (need license), 
  -hybrid, or 
  -fully managed cloud solution

  <img width="867" height="442" alt="image" src="https://github.com/user-attachments/assets/a85445f8-1abf-43c3-90dc-18df5337d5a4" />

  - vpc 
  
  - Kubernetes Cluster : 
    
  - Ingress : (Amazon ALB and a URL for users to access the Airbyte UI or make API requests.) 
  
  - Object Storage : (	Amazon S3 bucket with two directories for log and state storage.) 
  
  - Dedicated Database :	Amazon RDS Postgres with at least one read replica.

  - External Secrets Manager :	Amazon Secrets Manager for storing connector secrets.

### kubectl create namespace airbyte

### Creating a Kubernetes Secret
      apiVersion: v1
      kind: Secret
      metadata:
        name: airbyte-config-secrets
      type: Opaque
      stringData:
        # Enterprise License Key
        license-key: ## e.g. xxxxx.yyyyy.zzzzz
      
        # Database Secrets
        database-host: ## e.g. database.internal
        database-port: ## e.g. 5432
        database-name: ## e.g. airbyte
        database-user: ## e.g. airbyte
        database-password: ## e.g. password
      
        # Instance Admin
        instance-admin-email: ## e.g. admin@company.example
        instance-admin-password: ## e.g. password
      
        # SSO OIDC Credentials
        client-id: ## e.g. e83bbc57-1991-417f-8203-3affb47636cf
        client-secret: ## e.g. wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
      
        # AWS S3 Secrets
        s3-access-key-id: ## e.g. AKIAIOSFODNN7EXAMPLE
        s3-secret-access-key: ## e.g. wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
      
        # Azure Blob Storage Secrets
        azure-blob-store-connection-string: ## DefaultEndpointsProtocol=https;AccountName=azureintegration;AccountKey=wJalrXUtnFEMI/wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY/wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY==;EndpointSuffix=core.windows.net
      
        # AWS Secret Manager
        aws-secret-manager-access-key-id: ## e.g. AKIAIOSFODNN7EXAMPLE
        aws-secret-manager-secret-access-key: ## e.g. wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
      
        # Azure Secret Manager
        azure-key-vault-client-id: ## 3fc863e9-4740-4871-bdd4-456903a04d4e
        azure-key-vault-client-secret: ## KWP6egqixiQeQoKqFZuZq2weRbYoVxMH

### You can also use kubectl to create the secret directly from the CLI:
        kubectl create secret generic airbyte-config-secrets \
        --from-literal=license-key='' \
        --from-literal=database-host='' \
        --from-literal=database-port='' \
        --from-literal=database-name='' \
        --from-literal=database-user='' \
        --from-literal=database-password='' \
        --from-literal=instance-admin-email='' \
        --from-literal=instance-admin-password='' \
        --from-literal=s3-access-key-id='' \
        --from-literal=s3-secret-access-key='' \
        --from-literal=aws-secret-manager-access-key-id='' \
        --from-literal=aws-secret-manager-secret-access-key='' \
        --namespace airbyte


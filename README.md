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

## Installation Steps
          Step 1: Add Airbyte Helm Repository
          helm repo add airbyte https://airbytehq.github.io/helm-charts
          helm repo update
          helm search repo airbyte

          Step 2: Configure your Deployment
          created  values.yaml
          global:
          edition: enterprise
          airbyteUrl: # e.g. https://airbyte.company.example
          auth:
            instanceAdmin:
              firstName: ## First name of admin user.
              lastName: ## Last name of admin user.
            identityProvider:
              type: oidc
              secretName: airbyte-config-secrets ## Name of your Kubernetes secret.
              oidc:
                domain: ## e.g. company.example
                appName: ## e.g. airbyte
                clientIdSecretKey: client-id
                clientSecretSecretKey: client-secret
          
          Step3:Configuring the Airbyte Database
          postgresql:
            enabled: false
          global: 
            database:
              # -- Secret name where database credentials are stored
              secretName: "" # e.g. "airbyte-config-secrets"
              # -- The database host
              host: ""
              # -- The database port
              port:
              # -- The database name - this key used to be "database" in Helm chart 1.0
              name: ""
          
              # Use EITHER user or userSecretKey, but not both
              # -- The database user
              user: ""
              # -- The key within `secretName` where the user is stored
              userSecretKey: "" # e.g. "database-user"
          
              # Use EITHER password or passwordSecretKey, but not both
              # -- The database password
              password: ""
              # -- The key within `secretName` where the password is stored
              passwordSecretKey: "" # e.g."database-password"

          Step4: Configuring External Logging
          global:
            storage:
              secretName: ""
              type: minio # default storage is minio. Set to s3, gcs, or azure, according to what you use.
          
              bucket:
                log: airbyte-bucket
                state: airbyte-bucket
                workloadOutput: airbyte-bucket
                activityPayload: airbyte-bucket
              s3:
                region: "" ## e.g. us-east-1
                authenticationType: credentials ## Use "credentials" or "instanceProfile"
                accessKeyId: ""
                secretAccessKey: ""

          Step5: Configuring External Connector Secret Management
          global:
            secretsManager:
              enabled: false
              type: AWS_SECRET_MANAGER
              secretName: "airbyte-config-secrets"
              # Set ONE OF the following groups of configurations, based on your configuration in global.secretsManager.type.
              awsSecretManager:
                region: <aws-region>
                authenticationType: credentials ## Use "credentials" or "instanceProfile"
                tags: ## Optional - You may add tags to new secrets created by Airbyte.
                - key: ## e.g. team
                    value: ## e.g. deployments
                  - key: business-unit
                    value: engineering
                kms: ## Optional - ARN for KMS Decryption.

          Step6-1: Set up ingress in Airbyte from values.yaml
          ingress:
            enabled: true
            className: "nginx"  # Specify your ingress class
            annotations: {}
              # Add any ingress-specific annotations here
            hosts:
              - host: airbyte.example.com  # Replace with your domain
                paths:
                  - path: /auth
                    pathType: Prefix
                    backend: keycloak  # For Keycloak authentication (if using OIDC)
                  - path: /
                    pathType: Prefix
                    backend: server  # Routes to airbyte-server
                  - path: /connector-builder
                    pathType: Prefix
                    backend: connector-builder-server  # Required for connector builder
            tls: []
              # Optionally configure TLS
              # - secretName: airbyte-tls
              #   hosts:
              #     - airbyte.example.com

           Step6-2: Set up ingress in Airbyte (bring your own ingress) NGINX
              apiVersion: networking.k8s.io/v1
              kind: Ingress
              metadata:
                name: # ingress name, example: enterprise-demo
                annotations:
                  nginx.ingress.kubernetes.io/ssl-redirect: "false"
              spec:
                ingressClassName: nginx
                rules:
                  - host: airbyte.example.com # replace with your host
                    http:
                      paths:
                        - backend:
                            service:
                              # format is ${RELEASE_NAME}-airbyte-keycloak-svc 
                              name: airbyte-enterprise-airbyte-keycloak-svc 
                              port: 
                                number: 8180 
                          path: /auth
                          pathType: Prefix
                        - backend:
                            service:
                              # format is ${RELEASE_NAME}-airbyte-connector-builder-server-svc
                              name: airbyte-enterprise-airbyte-connector-builder-server-svc
                              port:
                                number: 80 # service port, example: 8080
                          path: /api/v1/connector_builder/
                          pathType: Prefix
                        - backend:
                            service:
                              # format is ${RELEASE_NAME}-airbyte-server-svc
                              name: airbyte-enterprise-airbyte-server-svc
                              port:
                                number: 8001 # service port, example: 8080
                          path: /
                          pathType: Prefix
                         
          Step6-3: Set up ingress in Airbyte (bring your own ingress) ALB
            apiVersion: networking.k8s.io/v1
            kind: Ingress
            metadata:
              name: airbyte-ingress # ingress name, e.g. airbyte-production-ingress
              annotations:
                # Specifies that the Ingress should use an AWS ALB.
                kubernetes.io/ingress.class: "alb"
                # Redirects HTTP traffic to HTTPS.
                alb.ingress.kubernetes.io/ssl-redirect: "443"
                # Creates an internal ALB, which is only accessible within your VPC or through a VPN.
                alb.ingress.kubernetes.io/scheme: internal
                # Specifies the ARN of the SSL certificate managed by AWS ACM, essential for HTTPS.
                alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-x:xxxxxxxxx:certificate/xxxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxx
                # Sets the idle timeout value for the ALB.
                alb.ingress.kubernetes.io/load-balancer-attributes: idle_timeout.timeout_seconds=30
                # [If Applicable] Specifies the VPC subnets and security groups for the ALB
                # alb.ingress.kubernetes.io/subnets: '' e.g. 'subnet-12345, subnet-67890'
                # alb.ingress.kubernetes.io/security-groups: <SECURITY_GROUP>
            spec:
              rules:
                - host: airbyte.example.com # replace with your host
                  http:
                    paths:
                      - backend:
                          service:
                            name: airbyte-enterprise-airbyte-keycloak-svc
                            port:
                              number: 8180
                        path: /auth
                        pathType: Prefix
                      - backend:
                          service:
                            name: airbyte-enterprise-airbyte-connector-builder-server-svc
                            port:
                              number: 80
                        path: /api/v1/connector_builder/
                        pathType: Prefix
                      - backend:
                          service:
                            name: airbyte-enterprise-airbyte-server-svc
                            port:
                              number: 8001
                        path: /
                        pathType: Prefix

          IAM POLICY FOR SERVICE ACCOUNT
          {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Action": [
                            "iam:CreateServiceLinkedRole"
                        ],
                        "Resource": "*",
                        "Condition": {
                            "StringEquals": {
                                "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
                            }
                        }
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "ec2:DescribeAccountAttributes",
                            "ec2:DescribeAddresses",
                            "ec2:DescribeAvailabilityZones",
                            "ec2:DescribeInternetGateways",
                            "ec2:DescribeVpcs",
                            "ec2:DescribeVpcPeeringConnections",
                            "ec2:DescribeSubnets",
                            "ec2:DescribeSecurityGroups",
                            "ec2:DescribeInstances",
                            "ec2:DescribeNetworkInterfaces",
                            "ec2:DescribeTags",
                            "ec2:GetCoipPoolUsage",
                            "ec2:DescribeCoipPools",
                            "ec2:GetSecurityGroupsForVpc",
                            "ec2:DescribeIpamPools",
                            "ec2:DescribeRouteTables",
                            "elasticloadbalancing:DescribeLoadBalancers",
                            "elasticloadbalancing:DescribeLoadBalancerAttributes",
                            "elasticloadbalancing:DescribeListeners",
                            "elasticloadbalancing:DescribeListenerCertificates",
                            "elasticloadbalancing:DescribeSSLPolicies",
                            "elasticloadbalancing:DescribeRules",
                            "elasticloadbalancing:DescribeTargetGroups",
                            "elasticloadbalancing:DescribeTargetGroupAttributes",
                            "elasticloadbalancing:DescribeTargetHealth",
                            "elasticloadbalancing:DescribeTags",
                            "elasticloadbalancing:DescribeTrustStores",
                            "elasticloadbalancing:DescribeListenerAttributes",
                            "elasticloadbalancing:DescribeCapacityReservation"
                        ],
                        "Resource": "*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "cognito-idp:DescribeUserPoolClient",
                            "acm:ListCertificates",
                            "acm:DescribeCertificate",
                            "iam:ListServerCertificates",
                            "iam:GetServerCertificate",
                            "waf-regional:GetWebACL",
                            "waf-regional:GetWebACLForResource",
                            "waf-regional:AssociateWebACL",
                            "waf-regional:DisassociateWebACL",
                            "wafv2:GetWebACL",
                            "wafv2:GetWebACLForResource",
                            "wafv2:AssociateWebACL",
                            "wafv2:DisassociateWebACL",
                            "shield:GetSubscriptionState",
                            "shield:DescribeProtection",
                            "shield:CreateProtection",
                            "shield:DeleteProtection"
                        ],
                        "Resource": "*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "ec2:AuthorizeSecurityGroupIngress",
                            "ec2:RevokeSecurityGroupIngress"
                        ],
                        "Resource": "*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "ec2:CreateSecurityGroup"
                        ],
                        "Resource": "*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "ec2:CreateTags"
                        ],
                        "Resource": "arn:aws:ec2:*:*:security-group/*",
                        "Condition": {
                            "StringEquals": {
                                "ec2:CreateAction": "CreateSecurityGroup"
                            },
                            "Null": {
                                "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                            }
                        }
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "ec2:CreateTags",
                            "ec2:DeleteTags"
                        ],
                        "Resource": "arn:aws:ec2:*:*:security-group/*",
                        "Condition": {
                            "Null": {
                                "aws:RequestTag/elbv2.k8s.aws/cluster": "true",
                                "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                            }
                        }
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "ec2:AuthorizeSecurityGroupIngress",
                            "ec2:RevokeSecurityGroupIngress",
                            "ec2:DeleteSecurityGroup"
                        ],
                        "Resource": "*",
                        "Condition": {
                            "Null": {
                                "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                            }
                        }
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "elasticloadbalancing:CreateLoadBalancer",
                            "elasticloadbalancing:CreateTargetGroup"
                        ],
                        "Resource": "*",
                        "Condition": {
                            "Null": {
                                "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                            }
                        }
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "elasticloadbalancing:CreateListener",
                            "elasticloadbalancing:DeleteListener",
                            "elasticloadbalancing:CreateRule",
                            "elasticloadbalancing:DeleteRule"
                        ],
                        "Resource": "*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "elasticloadbalancing:AddTags",
                            "elasticloadbalancing:RemoveTags"
                        ],
                        "Resource": [
                            "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*",
                            "arn:aws:elasticloadbalancing:*:*:loadbalancer/net/*/*",
                            "arn:aws:elasticloadbalancing:*:*:loadbalancer/app/*/*"
                        ],
                        "Condition": {
                            "Null": {
                                "aws:RequestTag/elbv2.k8s.aws/cluster": "true",
                                "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                            }
                        }
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "elasticloadbalancing:AddTags",
                            "elasticloadbalancing:RemoveTags"
                        ],
                        "Resource": [
                            "arn:aws:elasticloadbalancing:*:*:listener/net/*/*/*",
                            "arn:aws:elasticloadbalancing:*:*:listener/app/*/*/*",
                            "arn:aws:elasticloadbalancing:*:*:listener-rule/net/*/*/*",
                            "arn:aws:elasticloadbalancing:*:*:listener-rule/app/*/*/*"
                        ]
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "elasticloadbalancing:ModifyLoadBalancerAttributes",
                            "elasticloadbalancing:SetIpAddressType",
                            "elasticloadbalancing:SetSecurityGroups",
                            "elasticloadbalancing:SetSubnets",
                            "elasticloadbalancing:DeleteLoadBalancer",
                            "elasticloadbalancing:ModifyTargetGroup",
                            "elasticloadbalancing:ModifyTargetGroupAttributes",
                            "elasticloadbalancing:DeleteTargetGroup",
                            "elasticloadbalancing:ModifyListenerAttributes",
                            "elasticloadbalancing:ModifyCapacityReservation",
                            "elasticloadbalancing:ModifyIpPools"
                        ],
                        "Resource": "*",
                        "Condition": {
                            "Null": {
                                "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                            }
                        }
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "elasticloadbalancing:AddTags"
                        ],
                        "Resource": [
                            "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*",
                            "arn:aws:elasticloadbalancing:*:*:loadbalancer/net/*/*",
                            "arn:aws:elasticloadbalancing:*:*:loadbalancer/app/*/*"
                        ],
                        "Condition": {
                            "StringEquals": {
                                "elasticloadbalancing:CreateAction": [
                                    "CreateTargetGroup",
                                    "CreateLoadBalancer"
                                ]
                            },
                            "Null": {
                                "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                            }
                        }
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "elasticloadbalancing:RegisterTargets",
                            "elasticloadbalancing:DeregisterTargets"
                        ],
                        "Resource": "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "elasticloadbalancing:SetWebAcl",
                            "elasticloadbalancing:ModifyListener",
                            "elasticloadbalancing:AddListenerCertificates",
                            "elasticloadbalancing:RemoveListenerCertificates",
                            "elasticloadbalancing:ModifyRule",
                            "elasticloadbalancing:SetRulePriorities"
                        ],
                        "Resource": "*"
                    }
                ]
            }

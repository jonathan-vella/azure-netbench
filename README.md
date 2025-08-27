# Azure Network Internal Zone Latency Benchmark Dashboard

A comprehensive platform for testing and analyzing network performance between Azure Availability Zones across different regions worldwide.  
Current deployment: https://latency.azure.cloud63.fr/

## 🎯 Purpose

This project automatically measures network latency and bandwidth between Azure Availability Zones across 25+ global regions. It provides valuable insights into Azure's internal network performance to optimize multi-zone application deployments.

## 🏗️ Architecture

![Architecture High Level Diagram](images/archi-hld.png)  
For each region, one server VM and one client VM are deployed on each physical zone.
Client VMs initiate the tests to each server one by one and send results to the storage account table.
To avoid concurrent tests, each client is provisioned with a delay of 60 seconds.

![Architecture Low Level Diagram](images/archi-lld.png)  
Access to storage account is done using a managed identity and through a Private Endpoint.

### Test Infrastructure
- **Test VMs**: Automatic deployment of Linux virtual machines across 3 availability zones
- **Network Testing**: Uses `qperf` to measure TCP latency and bandwidth (`iperf3` is deployed but not used for the moment)
- **Data Storage**: Azure Table Storage for results persistence
- **Automation**: GitHub Actions for two hour test execution

### Visualization Dashboard
- **Frontend**: Static web application with interactive visualizations
- **Backend**: Azure Functions for data access API
- **Hosting**: Azure Static Web Apps

## 📊 Collected Metrics

### Network Latency
- **Intra-zone**: Latency between VMs in the same availability zone
- **Inter-zone**: Latency between VMs in different availability zones
- **Alert Thresholds**: 
  - Intra-zone > 500 μs (considered abnormal)
  - Inter-zone > 2000 μs (considered abnormal)

### Bandwidth
- TCP tests with `qperf` over 10 seconds
- Measurements in Gb/sec between all zone pairs
- Note: Bandwidth depends on VM types and is not the primary focus as the VM sizes can vary significantly between regions.

## 🌍 Tested Regions

The project covers 25+ Azure regions:

### Americas
- East US, East US 2, Central US, South Central US
- West US 2, West US 3, Canada Central
- Brazil South, Chile Central, Mexico Central

### Europe
- North Europe, West Europe, France Central
- Germany West Central, Sweden Central, Switzerland North
- UK South, Italy North, Spain Central, Norway East, Poland Central

### Asia-Pacific
- East Asia, Southeast Asia, Japan East, Japan West
- Korea Central, Australia East, New Zealand North
- Central India, Indonesia Central, Malaysia West

### Middle East & Africa
- UAE North, Qatar Central, Israel Central, South Africa North

## 🚀 Quick Start

### Prerequisites
- Azure account with Contributor permissions
- GitHub repository (for workflows)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [Terraform](https://www.terraform.io/downloads.html) >= 1.0

### GitHub Secrets Configuration

You can use the script `setup_github_secrets.sh` to configure the required GitHub secrets.

#### Azure Authentication
```bash
AZURE_SUBSCRIPTION_ID     # Your Azure subscription ID
AZURE_CLIENT_ID           # Service principal ID
AZURE_CLIENT_SECRET       # Service principal secret  
AZURE_TENANT_ID           # Azure tenant ID
```

#### Terraform Backend
```bash
TERRAFORM_STATE_RG              # Resource group for Terraform state
TERRAFORM_STATE_SA              # Storage account for Terraform state
TERRAFORM_STATE_CONTAINER       # Container for Terraform state
TERRAFORM_STATE_SUBSCRIPTION_ID # Storage account subscription
```

### Service Principal Creation

```bash
# Create the service principal
az ad sp create-for-rbac --name "azure-netbench-sp" \
  --role "Contributor" \
  --scopes "/subscriptions/YOUR_SUBSCRIPTION_ID"

# The command returns the values for your GitHub secrets
```

## ⚡ Running Tests

### Automatic (Recommended)
GitHub Actions main workflow run automatically every two hours.
This workflow starts the testing process for all regions.

### Manual
```bash
# Via GitHub CLI
gh workflow run "France Central Network Benchmark"

# Or via GitHub Actions interface
```

### Local (for development)
```bash
# Deploy a specific region
terraform init
terraform plan -var-file="tfvars/francecentral.tfvars"
terraform apply -var-file="tfvars/francecentral.tfvars"

# Wait 5 minutes for tests, then cleanup
terraform destroy -var-file="tfvars/francecentral.tfvars"
```

## 📊 Visualization Dashboard

🚀 **Live Demo**: [https://latency.azure.cloud63.fr/](https://latency.azure.cloud63.fr/)

### Dashboard Deployment

```bash
cd dashboard
azd auth login
azd init
azd up
```

### Dashboard Features
- **Real-time Visualizations**: Latency and bandwidth charts
- **Advanced Filters**: By region, availability zone, time period
- **Time Series Analysis**: Performance evolution over time
- **Data Export**: CSV export of filtered results
- **Visual Alerts**: Highlighting of abnormal results

## 📁 Project Structure

```
azure-netbench/
├── modules/netperf/              # Terraform module for testing
│   ├── main.tf                   # Main infrastructure
│   ├── cloud-init-server.txt     # Test server configuration
│   ├── cloud-init-client.txt     # Test client configuration
│   └── variables.tf              # Module variables
├── tfvars/                       # Region-specific configuration
│   ├── francecentral.tfvars      # France Central config
│   ├── eastus.tfvars            # East US config
│   └── ...                      # Other regions
├── dashboard/                    # Visualization application
│   ├── api/                     # Azure Functions
│   ├── web/                     # Static frontend
│   └── infra/                   # Terraform infrastructure
├── .github/workflows/            # GitHub Actions
│   ├── europe-region.yml        # Europe regions workflow
│   └── ...                      # Other workflows
├── main.tf                      # Root Terraform configuration
└── availabilityZoneMappings.json # Physical/logical zone mapping
```

## 🔧 Advanced Configuration

### Test Customization

#### Modify VM Types
```terraform
# In tfvars/[region].tfvars
benchmark = {
  "francecentral" = "Standard_D8as_v5"  
}
```

#### Modify Test Frequency
```yaml
# In .github/workflows/all-regions.yml
schedule:
  - cron: '0 */3 * * *'  # Every 3 hours
```

### Monitoring and Logs

#### GitHub Actions Logs
- Check the "Actions" tab of your repository
- Each job displays detailed Terraform logs
- Deployment failures don't prevent resource destruction

#### Test VM Logs
Tests run automatically via cloud-init and send their results to Azure Table Storage.

## 🚨 Troubleshooting

### Authentication Failures
- Verify that all Azure secrets are correctly configured
- Ensure the service principal has `Contributor` permissions

### Terraform Backend Failures
- Verify that the storage account and container exist
- Check the names in the `TERRAFORM_STATE_*` secrets

### Deployment Failures
- Check job logs to see Terraform errors
- Verify Azure quotas in target regions
- Ensure VM sizes are available in the regions

### Missing Data in Dashboard
- Check managed identity permissions on the storage account
- Verify Azure Functions environment variables configuration

## 📈 Results Analysis

### Normal Results
- **Intra-zone latency**: < 500 μs
- **Inter-zone latency**: < 2000 μs
- **Bandwidth**: Variable depending on VM types and regions

### Abnormal Results
Results exceeding normal thresholds are:
- Highlighted in the dashboard

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-feature`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/new-feature`)
5. Create a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

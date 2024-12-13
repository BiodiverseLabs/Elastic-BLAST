# Elastic BLAST Search on AWS

## Overview
This repository provides a comprehensive guide to setting up and running an Elastic BLAST search on Amazon Web Services (AWS). Elastic BLAST is a cloud-native implementation of BLAST (Basic Local Alignment Search Tool) designed for distributed, high-performance searches of large datasets.

## Quick Guide (for future runs)
My two primary files for quick copy-and-paste creation of an environment and implementation of the scripts. Download and modify:
1. Elastic-BLAST_Install.txt
2. biodiverse_eblast.ini

## Prerequisites
Before proceeding, ensure you have the following:

- **AWS Account**: A valid AWS account with administrative permissions.
- **Elastic BLAST CLI**: Installed on the AWS CloudShell environment. [Installation Guide](https://github.com/ncbi/elastic-blast/wiki/Elastic-BLAST-Installation)
- **IAM Permissions**: Permissions to create and manage EC2 instances, S3 buckets, and other necessary AWS resources.
- **S3 Bucket**: An S3 bucket to store query data and results.
- **BLAST Database**: Preformatted BLAST databases stored in an accessible S3 bucket or on the AWS Public Dataset Program.
- **AWS Quota** Request an update of the AWS service quota to 256 for: Service Quotas; AWS services; Amazon Elastic Compute Cloud (Amazon EC2); All Standard (A, C, D, H, I, M, R, T, Z) Spot Instance Requests

## Step-by-Step Instructions

### 1. Access AWS CloudShell
1. Log in to your AWS Management Console.
2. Launch CloudShell from the AWS Console by clicking the CloudShell icon in the top-right corner.

### 2. Configure AWS Environment in CloudShell
1. Verify that AWS CLI is pre-installed and configured:
    ```bash
    aws configure
    ```
    Provide your AWS access key, secret key, region, and output format when prompted (if not already configured).

2. Verify your AWS configuration:
    ```bash
    aws s3 ls
    ```
### 3. Allow for Automatic Cleanup
    ```bash
    aws-create-elastic-blast-janitor-role.sh
    ```
    
### 4. Prepare Input Data
1. Upload your query file(s) to an S3 bucket from CloudShell:
    ```bash
    aws s3 cp /path/to/query_file.fasta s3://your-bucket-name/query_file.fasta
    ```

2. Note the S3 path to your BLAST database if not using the public dataset.

### 5. Set Up Elastic BLAST Environment
Before installing Elastic BLAST, set up a clean Python virtual environment in CloudShell:

```bash
[ -d .elb-venv ] && rm -fr .elb-venv 
python3 -m venv .elb-venv
source .elb-venv/bin/activate
```

Then, install Elastic BLAST within the virtual environment:
```bash
pip install elastic-blast
```

### 6. Configure Elastic BLAST
1. Create a configuration file (e.g., `elastic-blast-config.ini`) directly in CloudShell:
    ```bash
    nano elastic-blast-config.ini
    ```

    Add the following content to the file:
    ```ini
    [cloud-provider]
    provider = AWS
    region = us-east-1

    [cluster]
    instance-type = m5.large
    num-instances = 10

    [blast]
    program = blastn
    db = s3://your-bucket-name/blast-database
    queries = s3://your-bucket-name/query_file.fasta

    [output]
    bucket = s3://your-bucket-name/blast-results
    ```

2. Replace placeholder values with your AWS resources and preferences.

### 7. Run Elastic BLAST
1. Start the Elastic BLAST job:
    ```bash
    elastic-blast submit -c elastic-blast-config.ini
    ```

2. Monitor the job:
    ```bash
    elastic-blast status -c elastic-blast-config.ini
    ```

### 8. Retrieve Results
Once the job completes, retrieve your results from the specified S3 bucket:
```bash
aws s3 cp s3://your-bucket-name/blast-results/ ./blast-results --recursive
```

## Cleanup
To avoid incurring unnecessary charges, clean up your resources:
1. Delete the S3 bucket or its contents:
    ```bash
    aws s3 rm s3://your-bucket-name/ --recursive
    ```
2. Terminate the Elastic BLAST cluster:
    ```bash
    elastic-blast delete -c elastic-blast-config.ini
    ```

## Troubleshooting
- **Permission Issues**: Ensure your IAM user/role has the required permissions for Elastic BLAST and AWS resources.
- **Configuration Errors**: Double-check your `elastic-blast-config.ini` file for typos or misconfigurations.
- **Resource Limits**: Adjust the instance type or number of instances in your configuration if the job fails due to insufficient resources.

## Resources
- [Elastic BLAST Documentation](https://github.com/ncbi/elastic-blast/wiki)
- [AWS Documentation](https://docs.aws.amazon.com/)
- [BLAST+ User Manual](https://www.ncbi.nlm.nih.gov/books/NBK279690/)

## License
This project is licensed under the MIT License. See the LICENSE file for details.

## License
This project is licensed under the MIT License. See the LICENSE file for details.


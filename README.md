# Amazon-CodeGuru-Security-Step-by-Step-Hands-On-Guide
![image](https://github.com/user-attachments/assets/ed4c86d5-451c-40a5-8eca-0476721ddffb)
Amazon CodeGuru Security is a static application security testing (SAST) tool that combines machine learning (ML) and automated reasoning to identify vulnerabilities in your code, provide recommendations on how to fix the identified vulnerabilities, and track the status of the vulnerabilities until closure. 


# Amazon CodeGuru Security: Step-by-Step Hands-On Guide

This comprehensive guide will walk you through setting up and using Amazon CodeGuru Security to identify security vulnerabilities and follow best practices in your code.

## Introduction to Amazon CodeGuru Security

Amazon CodeGuru Security is a developer tool that uses machine learning to detect security vulnerabilities in your code and provides recommendations for remediation. It can identify issues such as:

- Authentication problems
- Input validation flaws
- Encryption errors
- Resource leaks
- Code injection vulnerabilities
- Weak cryptography practices

## Prerequisites

Before starting, ensure you have:

- AWS account with appropriate permissions
- AWS CLI installed and configured
- Git installed
- A code repository (GitHub, Bitbucket, AWS CodeCommit, or local repository)
- Java, Python, or JavaScript/TypeScript code to analyze

## Step 1: Set Up AWS Permissions

First, create an IAM user or role with appropriate permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "codeguru-security:*",
                "s3:CreateBucket",
                "s3:PutObject",
                "s3:GetObject",
                "s3:ListBucket",
                "codecommit:GetRepository",
                "codecommit:GetBranch",
                "codecommit:GetCommit",
                "codecommit:ListRepositories"
            ],
            "Resource": "*"
        }
    ]
}
```

## Step 2: Install and Configure AWS CLI

If you haven't already, install AWS CLI:

```bash
# For Linux/macOS
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# For Windows
# Download and run the installer from https://awscli.amazonaws.com/AWSCLIV2.msi
```

Configure AWS CLI with your credentials:

```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, region (e.g., us-east-1), and output format (json)
```

## Step 3: Set Up a Repository for Analysis

You can use an existing repository or create a new one for this guide. Here's how to create a sample repository with vulnerability examples:

```bash
mkdir codeguru-security-demo
cd codeguru-security-demo
git init

# Create a sample Java file with security issues
cat > VulnerableCode.java << 'EOL'
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;

public class VulnerableCode {
    public static void main(String[] args) {
        try {
            // Vulnerability 1: Hardcoded credentials
            String password = "admin123";
            Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/test", "admin", password);
            
            // Vulnerability 2: SQL Injection
            String userId = args[0];
            Statement stmt = conn.createStatement();
            stmt.execute("SELECT * FROM users WHERE id = " + userId);
            
            // Vulnerability 3: Weak cryptography
            String key = "weakkey12345678";
            javax.crypto.spec.SecretKeySpec secretKey = new javax.crypto.spec.SecretKeySpec(key.getBytes(), "AES");
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
EOL

git add .
git commit -m "Initial commit with sample code"
```

## Step 4: Create an Amazon S3 Bucket for Code Analysis

CodeGuru Security requires an S3 bucket to store analysis artifacts:

```bash
aws s3api create-bucket --bucket codeguru-security-demo-$RANDOM --region us-east-1
```

Note the bucket name for the next step.

## Step 5: Create a CodeGuru Security Scan

You can create a scan using either the AWS Management Console or AWS CLI:

### Using AWS Management Console:

1. Open the AWS Management Console
2. Navigate to Amazon CodeGuru Security
3. Click "Create scan"
4. Choose your scan source (GitHub, Bitbucket, AWS CodeCommit, or upload code)
5. Configure the scan settings
6. Start the scan

### Using AWS CLI:

```bash
# Create a scan
aws codeguru-security create-scan --name "DemoScan" \
  --analysis-type "Security" \
  --resource-path "local://$PWD" \
  --s3-bucket-name "YOUR_S3_BUCKET_NAME" \
  --scan-name "SecurityScan-$(date +%Y%m%d%H%M%S)" \
  --scan-type "Express"
```

This command will output a scan ID. Note this ID for the next steps.

## Step 6: Monitor Scan Progress

Check the status of your scan:

```bash
aws codeguru-security get-scan --scan-name "YOUR_SCAN_NAME"
```

You can also monitor the progress in the AWS Management Console.

## Step 7: Review Security Findings

Once the scan is complete, you can review the findings:

### Using AWS Management Console:

1. Navigate to Amazon CodeGuru Security
2. Select your scan
3. View the list of findings
4. Click on individual findings to see detailed recommendations

### Using AWS CLI:

```bash
# Get all findings from your scan
aws codeguru-security get-findings --scan-name "YOUR_SCAN_NAME"

# Get detailed information about a specific finding
aws codeguru-security get-finding --scan-name "YOUR_SCAN_NAME" --finding-id "FINDING_ID"
```

## Step 8: Export and Share Findings

Export your findings to share with your team:

```bash
# Export findings to a JSON file
aws codeguru-security get-findings --scan-name "YOUR_SCAN_NAME" \
  --query "findings" --output json > codeguru-findings.json
```

You can also generate reports from the AWS Console.

## Step 9: Remediate Security Issues

Based on the recommendations provided by CodeGuru Security, fix the security issues in your code:

For our example vulnerabilities:

1. **Hardcoded credentials**: Use AWS Secrets Manager or environment variables instead.
2. **SQL Injection**: Use parameterized queries.
3. **Weak cryptography**: Use stronger algorithms and keys.

Here's how you might fix our example code:

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import software.amazon.awssdk.services.secretsmanager.SecretsManagerClient;
import software.amazon.awssdk.services.secretsmanager.model.GetSecretValueRequest;

public class FixedCode {
    public static void main(String[] args) {
        try {
            // Fixed 1: Use AWS Secrets Manager for credentials
            SecretsManagerClient secretsClient = SecretsManagerClient.create();
            GetSecretValueRequest request = GetSecretValueRequest.builder()
                .secretId("db-credentials")
                .build();
            String connectionString = secretsClient.getSecretValue(request).secretString();
            Connection conn = DriverManager.getConnection(connectionString);
            
            // Fixed 2: Use parameterized queries to prevent SQL Injection
            String userId = args[0];
            PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
            pstmt.setString(1, userId);
            pstmt.executeQuery();
            
            // Fixed 3: Use proper key generation
            KeyGenerator keyGen = KeyGenerator.getInstance("AES");
            keyGen.init(256); // Use appropriate key size
            SecretKey secretKey = keyGen.generateKey();
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Step 10: Set Up Continuous Security Scanning

Integrate CodeGuru Security into your CI/CD pipeline:

### For AWS CodePipeline:

1. Create a buildspec.yml file in your repository:

```yaml
version: 0.2

phases:
  build:
    commands:
      - aws codeguru-security create-scan --name "CICD-Scan" --analysis-type "Security" --resource-path "local://$CODEBUILD_SRC_DIR" --s3-bucket-name "YOUR_S3_BUCKET_NAME" --scan-name "SecurityScan-$(date +%Y%m%d%H%M%S)" --scan-type "Express"
  post_build:
    commands:
      - SCAN_RESULTS=$(aws codeguru-security get-findings --scan-name "SecurityScan-$(date +%Y%m%d%H%M%S)")
      - echo $SCAN_RESULTS
      - if echo $SCAN_RESULTS | grep -q '"severity": "High"'; then exit 1; fi

artifacts:
  files:
    - '**/*'
```

2. Configure your CodeBuild project to use this buildspec.yml
3. Add the CodeBuild project to your CodePipeline

### For GitHub Actions:

Create a `.github/workflows/codeguru-security.yml` file:

```yaml
name: CodeGuru Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Run CodeGuru Security Scan
      run: |
        aws codeguru-security create-scan \
          --name "GitHub-Scan" \
          --analysis-type "Security" \
          --resource-path "local://$GITHUB_WORKSPACE" \
          --s3-bucket-name "YOUR_S3_BUCKET_NAME" \
          --scan-name "SecurityScan-$GITHUB_SHA" \
          --scan-type "Express"
        
        # Wait for scan to complete
        SCAN_STATUS="InProgress"
        while [ "$SCAN_STATUS" == "InProgress" ]; do
          sleep 30
          SCAN_STATUS=$(aws codeguru-security get-scan --scan-name "SecurityScan-$GITHUB_SHA" --query "status" --output text)
        done
        
        # Check for high severity findings
        FINDINGS=$(aws codeguru-security get-findings --scan-name "SecurityScan-$GITHUB_SHA")
        echo "$FINDINGS"
        if echo "$FINDINGS" | grep -q '"severity": "High"'; then
          exit 1
        fi
```

## Step 11: Best Practices for Using CodeGuru Security

1. **Scan early and often**: Integrate security scanning into your development process.
2. **Prioritize findings**: Address high-severity findings first.
3. **Document exemptions**: If you decide not to fix certain findings, document the reasons.
4. **Educate your team**: Share CodeGuru Security findings to help developers learn about security best practices.
5. **Customize severity thresholds**: Configure your CI/CD pipeline to fail builds based on your organization's risk tolerance.

## Troubleshooting

Common issues and how to resolve them:

1. **Access Denied Errors**: Check IAM permissions for the user or role.
2. **Scan Failures**: Verify the S3 bucket permissions and repository access.
3. **Missing Findings**: Ensure your code is in a supported language (Java, Python, JavaScript/TypeScript).
4. **Long Scan Times**: For large repositories, consider scanning specific directories.

## Conclusion

Amazon CodeGuru Security provides powerful automation for identifying security vulnerabilities in your code. By integrating it into your development workflow, you can catch and fix security issues before they make it into production.

For more information, visit the [Amazon CodeGuru Security documentation](https://docs.aws.amazon.com/codeguru/latest/security-ug/what-is-codeguru-security.html).

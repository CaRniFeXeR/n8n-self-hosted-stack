# n8n Self-Hosted Stack on AWS

A comprehensive AWS CloudFormation template that automatically deploys a complete n8n workflow automation platform with additional services including Selenium WebDriver and n8n MCP Server integration.

## 🚀 Features

- **Fully Automated Deployment**: One-click deployment using AWS CloudFormation
- **n8n Workflow Automation**: Complete n8n instance with PostgreSQL database
- **Selenium WebDriver**: For web automation and testing workflows
- **n8n MCP Server**: AI assistant integration for workflow management
- **Caddy Reverse Proxy**: Automatic HTTPS with SSL certificates
- **Custom Domain Support**: Serve n8n on your own domain
- **Security**: Proper security groups and configurable secrets
- **Scalable**: Built on AWS infrastructure

## 📋 Prerequisites

Before deploying this stack, ensure you have:

1. **AWS Account**: With sufficient privileges to create CloudFormation stacks, EC2 instances, and other AWS resources
2. **S3 Bucket**: To hold the CloudFormation template file (AWS can create one for you, but having your own bucket avoids awkward auto-generated names)
3. **Domain Name**: A domain name where you want to host n8n (e.g., `n8n.yourdomain.com` or `automation.yourdomain.com`)
4. **DNS Provider**: Access to configure DNS records (Cloudflare, Route53, etc.) - Cloudflare is recommended
5. **n8n API Key**: You'll need to generate this in your n8n instance after deployment

## 🏗️ Architecture

This stack creates the following AWS resources:

- **EC2 Instance**: Amazon Linux 2023 with Docker and Docker Compose
- **Elastic IP**: Static IP address for your n8n instance
- **Security Group**: Configured for HTTP, HTTPS, SSH, n8n (5678), and Selenium (4444)
- **Lambda Function**: For automated SSH key pair creation and management
- **SSM Parameter Store**: Secure storage for SSH private keys

## 🚀 Quick Start

### The "Big Green Button" Approach

Following [Saurabh's philosophy](https://saurabh-sawhney.medium.com/self-hosting-the-workflow-automation-tool-n8n-on-aws-ec2-d4d2abf06282), this stack is designed as a "Big Green Button" - you press it and everything gets done automatically. No more manual setup steps, debugging, or configuration headaches.

### Step 1: Deploy the CloudFormation Stack

1. **Upload Template**: Upload the `aws-stack.yaml` file to an S3 bucket
2. **Create Stack**: Use the CloudFormation console to create a new stack
3. **Configure Parameters**:
   - `DomainName`: Your domain (e.g., `n8n.yourdomain.com` or `automation.yourdomain.com`)
   - `WebhookUrl`: Your webhook URL (e.g., `https://n8n.yourdomain.com/`)
   - `N8nEncryptionKey`: A secure random string for n8n encryption
   - `N8nJwtSecret`: A secure random string for JWT authentication
   - `N8nApiKey`: Your n8n API key (generate this after deployment)

### Step 2: Configure DNS (Critical Step)

**Important**: This step is best done immediately after initiating the CloudFormation stack to allow DNS resolution to settle down by the time Caddy is up and running.

1. **Get Elastic IP**: From the CloudFormation outputs, copy the Elastic IP address (available almost immediately)
2. **Create A Record**: In your DNS provider (like Cloudflare), create an A record pointing your domain to the Elastic IP
3. **DNS Settings**: Use "DNS only" setting, **NOT proxied** - this is crucial for Caddy to work properly
4. **Wait for Propagation**: DNS changes can take up to 48 hours (usually much faster)

### Step 3: Access Your n8n Instance

After approximately 10 minutes (the time taken for setup on the EC2), your n8n instance will be available at:
- **n8n Interface**: `https://your-domain.com`
- **Selenium WebDriver**: `http://elastic-ip:4444`

**Voila!** That's it - no more debugging, no more configuration headaches. Just like [Saurabh's experience](https://saurabh-sawhney.medium.com/self-hosting-the-workflow-automation-tool-n8n-on-aws-ec2-d4d2abf06282), this should work smoothly the first time around.

## 🔧 Configuration

### Environment Variables

The stack automatically configures the following environment variables:

```env
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8n
POSTGRES_DB=n8n
N8N_ENCRYPTION_KEY=your-encryption-key-here
N8N_USER_MANAGEMENT_JWT_SECRET=your-jwt-secret-here
N8N_PAYLOAD_SIZE_MAX=100
NODE_FUNCTION_ALLOW_EXTERNAL=node-fetch
WEBHOOK_URL=https://your-domain.com/
N8N_API_KEY=your-n8n-api-key-here
```

### Services Included

1. **n8n**: Main workflow automation platform
2. **PostgreSQL**: Database for n8n data
3. **Selenium**: WebDriver for web automation
4. **n8n MCP Server**: AI assistant integration
5. **Caddy**: Reverse proxy with automatic HTTPS

## 🔐 Security

- **SSH Access**: Private keys are stored securely in AWS SSM Parameter Store
- **Security Groups**: Only necessary ports are open (22, 80, 443, 5678, 4444)
- **HTTPS**: Automatic SSL certificates via Caddy
- **Configurable Secrets**: Encryption keys and JWT secrets can be customized

## 📊 Monitoring and Management

### CloudFormation Outputs

The stack provides the following outputs:
- `CaddyDomain`: Your n8n URL
- `SeleniumUrl`: Selenium WebDriver URL
- `ElasticIP`: Static IP address
- `McpServerInfo`: MCP server configuration details
- `KeyDownloadCommand`: AWS CLI command to download SSH key

### SSH Access

To access your EC2 instance via SSH:

```bash
# Download the SSH private key
aws ssm get-parameter --name /ec2/privateKey/n8n-automatic-EC2-PemFile --with-decryption --query Parameter.Value --output text > n8n-automatic-EC2-PemFile.pem

# Set proper permissions
chmod 400 n8n-automatic-EC2-PemFile.pem

# Connect to the instance
ssh -i n8n-automatic-EC2-PemFile.pem ec2-user@your-elastic-ip
```

## 🛠️ Troubleshooting

### Common Issues

1. **DNS Not Resolving**: Wait for DNS propagation or check your DNS configuration
2. **HTTPS Not Working**: Ensure your DNS A record is not proxied (DNS only) - this is crucial for Caddy to work properly
3. **Services Not Starting**: Check Docker logs: `docker compose logs`
4. **API Key Issues**: Generate a new API key in n8n settings
5. **Browser Security Warnings**: If you see "unsafe" warnings, ensure you're using HTTPS and proper DNS configuration

### Useful Commands

```bash
# Check service status
docker compose ps

# View logs
docker compose logs n8n
docker compose logs selenium
docker compose logs n8n-mcp-server

# Restart services
docker compose restart

# Update n8n
docker compose pull n8n
docker compose up -d n8n
```

## 💰 Cost Considerations

This stack creates the following billable resources:
- **EC2 Instance**: t4g.micro (~$8-12/month)
- **Elastic IP**: Free when attached to running instance
- **EBS Storage**: 30GB gp3 volume (~$3/month)
- **Data Transfer**: Varies based on usage

**Estimated Monthly Cost**: $10-15 USD

**Caution**: Since resources are created, AWS will start charging you accordingly. Be aware of this and delete the stack if you are no longer using the setup.

## 🧹 Cleanup

To avoid ongoing charges, delete the CloudFormation stack when you're done:

1. Go to AWS CloudFormation console
2. Select your stack
3. Click "Delete" and confirm

This will remove all created resources.

## 🤝 Contributing

This project is based on several open-source contributions. Feel free to submit issues or pull requests to improve the stack.

## 📄 License

This project is licensed under the MIT License.

## 🙏 Acknowledgments

This stack builds upon the work of several amazing developers and projects:

### Core Technologies
- **[n8n](https://n8n.io/)**: The powerful workflow automation platform that makes this all possible
- **[Docker](https://www.docker.com/)**: Containerization technology
- **[Caddy](https://caddyserver.com/)**: Modern web server with automatic HTTPS

### Original AWS Stack
- **[Saurabh Sawhney](https://saurabh-sawhney.medium.com/self-hosting-the-workflow-automation-tool-n8n-on-aws-ec2-d4d2abf06282)**: Created the original AWS CloudFormation template for n8n deployment and introduced the "Big Green Button" concept - the idea of automating all cumbersome setup steps into a single, smooth deployment. This stack is based on his excellent work and extends it with additional features while maintaining his philosophy of simplicity and automation.

### n8n MCP Server Integration
- **[Leonard Sellem](https://github.com/leonardsellem/n8n-mcp-server)**: Developed the n8n MCP server that enables AI assistants to interact with n8n workflows. This integration allows for powerful AI-driven workflow management and automation.

### Additional Features
- **Selenium Integration**: Added for web automation capabilities
- **Enhanced Security**: Improved security group configuration
- **Custom Environment**: Configurable environment variables and secrets

## 📞 Support

If you encounter issues or have questions:

1. Check the troubleshooting section above
2. Review the n8n documentation: https://docs.n8n.io/
3. Check the original AWS stack guide: https://saurabh-sawhney.medium.com/self-hosting-the-workflow-automation-tool-n8n-on-aws-ec2-d4d2abf06282
4. Review the MCP server documentation: https://github.com/leonardsellem/n8n-mcp-server

---

**Note**: This stack is designed for development and testing environments. For production use, consider additional security measures, monitoring, and backup strategies. 
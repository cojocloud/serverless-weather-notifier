# Serverless Weather Notifier with Terraform

This project implements a serverless weather notification system deployed on AWS using Terraform. It automatically fetches weather forecasts for a specified location and sends formatted email notifications to a list of recipients at scheduled times.

## Project Overview

The application consists of the following AWS services:

- **AWS Lambda**: Runs a Python function that fetches weather data from the AccuWeather API, processes it, and sends emails.
- **Amazon SES (Simple Email Service)**: Handles sending of email notifications.
- **Amazon CloudWatch Events**: Triggers the Lambda function on a scheduled basis using cron expressions.
- **IAM Roles and Policies**: Provides necessary permissions for Lambda to execute and send emails via SES.

The Lambda function retrieves a 12-hour weather forecast for a given location, formats the data into an HTML email template, and sends it to the specified email addresses.

## Architecture

1. CloudWatch Events rule triggers the Lambda function based on a cron schedule.
2. Lambda function calls the AccuWeather API to get weather data.
3. Data is processed and rendered using a Jinja2 HTML template.
4. SES sends the formatted email to recipients.
5. All infrastructure is provisioned and managed via Terraform.

## Prerequisites

Before deploying this project, ensure you have the following:

1. **AWS Account**: An active AWS account with appropriate permissions to create Lambda functions, CloudWatch rules, SES configurations, and IAM roles.

2. **Terraform**: Install Terraform on your local machine. Download from [terraform.io](https://www.terraform.io/downloads.html).

3. **AWS CLI**: Configure AWS CLI with your credentials. Run `aws configure` and set up your default profile.

4. **SES Verified Email**: 
   - Go to the AWS SES console.
   - Verify an email address that will be used as the sender (FROM_EMAIL).
   - Note: For production use, you may need to request SES production access.

5. **AccuWeather API Key**:
   - Register at [AccuWeather Developer Portal](https://developer.accuweather.com/).
   - Create an application and obtain your API key.

6. **Location ID**:
   - Use the AccuWeather Locations API to search by city name:
     ```bash
     curl "http://dataservice.accuweather.com/locations/v1/cities/search?apikey=YOUR_API_KEY&q=CITY_NAME"
     ```
   - Replace `CITY_NAME` with your city (e.g., `Sterling%20VA`).
   - Extract the `Key` value from the JSON response (e.g., `"Key": "123456"`).

## Step-by-Step Implementation Guide

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd serverless-weather-notifier-terraform
```

### Step 2: Configure Variables

Edit the `terraform.tfvars` file and fill in the required values:

```hcl
API_KEY = "your-accuweather-api-key"  # Your AccuWeather API key
EMAILS = "recipient1@example.com,recipient2@example.com"  # Comma-separated list of email addresses
FROM_EMAIL = "your-verified-ses-email@example.com"  # Verified SES email address
LOCATION_ID = "12345"  # AccuWeather location ID
EVENT_SCHEDULE = "35 10,17 * * ? *"  # Cron expression for CloudWatch Events (UTC) — default: 5:35 AM and 12:35 PM EST
```

**Note**: The `EVENT_SCHEDULE` uses AWS CloudWatch cron format and must be specified in **UTC**. The default triggers at 10:35 AM and 5:35 PM UTC, which corresponds to 5:35 AM and 12:35 PM EST. Adjust as needed.

### Step 3: Prepare the Deployment Script

Make the deployment script executable:

```bash
chmod +x deploy.sh
```

### Step 4: Deploy the Infrastructure

Run the deployment script:

```bash
./deploy.sh
```

This script will:
1. Navigate to the `lambda/` directory
2. Install Python dependencies from `requirements.txt` into the directory
3. Create a ZIP file (`lambda.zip`) containing the Lambda function code and dependencies
4. Return to the root directory
5. Run `terraform apply` to provision the AWS resources

Terraform will show you the planned changes. Review and type `yes` to proceed.

### Step 5: Verify Deployment

After successful deployment:

1. Check the AWS Lambda console to ensure the function is created.
2. Verify the CloudWatch Events rule is active.
3. Confirm SES is configured (check verified email addresses).
4. Monitor CloudWatch Logs for any Lambda execution errors.

### Step 6: Test the Notification

You can test the Lambda function manually:

1. Go to the AWS Lambda console.
2. Select the `weather-notifier` function.
3. Click "Test" and create a test event (empty JSON is fine).
4. Check the execution results and CloudWatch logs.

The function should send an email with the weather forecast to the specified recipients.

## Customization

### Changing the Schedule

Modify the `EVENT_SCHEDULE` variable in `terraform.tfvars`. Use AWS CloudWatch cron format (all times are **UTC**):
- `35 10,17 * * ? *` - Daily at 5:35 AM and 12:35 PM EST (10:35 and 17:35 UTC)
- `0 13 * * ? *` - Daily at 8:00 AM EST (13:00 UTC)
- `0 0/6 * * ? *` - Every 6 hours

### Modifying the Email Template

Edit `lambda/template.html` to customize the email appearance. The template uses Jinja2 syntax and receives weather data as `results`.

### Adding More Recipients

Update the `EMAILS` variable with additional comma-separated email addresses.

## Troubleshooting

### Common Issues

1. **SES Email Not Sending**:
   - Ensure the FROM_EMAIL is verified in SES.
   - Check if you're in SES sandbox mode (limits apply).

2. **Lambda Function Errors**:
   - Check CloudWatch Logs for the Lambda function.
   - Verify API_KEY and LOCATION_ID are correct.
   - Ensure AccuWeather API limits aren't exceeded.

3. **Terraform Apply Fails**:
   - Verify AWS credentials and permissions.
   - Check if required AWS services are available in your region.

4. **No Weather Data**:
   - Confirm LOCATION_ID is valid.
   - Check AccuWeather API key validity.

### Logs and Monitoring

- **Lambda Logs**: Available in CloudWatch Logs under `/aws/lambda/weather-notifier`
- **CloudWatch Events**: Check rule execution history
- **SES**: Monitor email sending metrics

## Cleanup

To destroy the infrastructure:

```bash
terraform destroy
```

Review the resources to be destroyed and confirm.

## Security Considerations

- Store API keys securely (consider AWS Secrets Manager for production).
- Use least-privilege IAM policies.
- Regularly rotate API keys.
- Monitor SES usage to avoid unexpected costs.

## Cost Estimation

- **Lambda**: ~$0.20/month for 2 daily executions
- **CloudWatch Events**: ~$1/month
- **SES**: ~$0.10 per 1000 emails
- **AccuWeather API**: Check their pricing (free tier available)

## Contributing

Feel free to submit issues and enhancement requests!

## License

See LICENSE.md for details.
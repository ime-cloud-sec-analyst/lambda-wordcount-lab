# lambda-wordcount-lab
AWS Lambda word-count challenge lab with S3 trigger and SNS notifications
# AWS Lambda Word Count Lab

This repository contains my solution for the AWS Lambda challenge lab:

- **Lambda function** in `lambda_function.py`  
- **Sample text file** `sample1.txt` (8 words)  
- **Screenshots** in the `screenshots/` directory

---

## Lab Objectives

1. Create a Python AWS Lambda function to count words in an S3-uploaded text file.  
2. Configure S3 to trigger the function on `.txt` file uploads.  
3. Use SNS to email the word count result.  

---

## Setup

1. **Create an SNS topic** named `WordCountTopic` and subscribe your email.  
2. **Create an S3 bucket** (e.g. `wordcount-lab-<yourinitials>`).  
3. **Deploy the Lambda function**:
   - Runtime: Python 3.9  
   - Execution role: `LambdaAccessRole`  
   - Environment variable:  
     ```
     SNS_TOPIC_ARN=arn:aws:sns:<region>:<account-id>:WordCountTopic
     ```
4. **Add an S3 trigger** on your bucket for **PUT** events with suffix `.txt`.

---

## Files

- **lambda_function.py**  
  Contains the handler code:
  ```python
  import boto3, os, urllib.parse

  s3 = boto3.client('s3')
  sns = boto3.client('sns')
  SNS_TOPIC_ARN = os.environ['SNS_TOPIC_ARN']

  def lambda_handler(event, context):
      rec = event['Records'][0]['s3']
      bucket = rec['bucket']['name']
      key = urllib.parse.unquote_plus(rec['object']['key'])
      text = s3.get_object(Bucket=bucket, Key=key)['Body'].read().decode('utf-8')
      count = len(text.split())
      msg = f"The word count in the {key} file is {count}."
      sns.publish(TopicArn=SNS_TOPIC_ARN, Subject="Word Count Result", Message=msg)
      return {'statusCode': 200, 'body': msg}

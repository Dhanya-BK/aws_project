# aws_project
AWS Lambda + S3 serverless image processing pipeline

A fully serverless image processing pipeline built on AWS, developed as part of my AWS Cloud internship project. The pipeline automatically resizes, compresses, and watermarks any image uploaded to an S3 bucket — with zero servers to manage.
User uploads image
        │
        ▼
  S3 Upload Bucket  ──(PUT event trigger)──▶  AWS Lambda Function
                                                      │
                                          Pillow: resize, compress,
                                              add watermark
                                                      │
                                                      ▼
                                          S3 Processed Bucket
                                        (processed-<filename>.jpg)

1.An image is uploaded to the upload S3 bucket.
2.The upload event automatically triggers an AWS Lambda function.
3.Lambda downloads the image, uses Pillow to resize it (max width 800px), add a text watermark, and compress it as JPEG.
4.The processed image is uploaded to a separate processed S3 bucket.

Tech Stack
-----------
AWS Lambda — Python 3.12 runtime, serverless compute
Amazon S3 — two buckets (upload + processed) for storage and event triggering
Pillow (PIL) — image resizing, format conversion, and watermarking, added via a Klayers Lambda layer
boto3 — AWS SDK for Python, used to read/write S3 objects
AWS IAM — custom execution role scoped for S3 access and CloudWatch logging.

How It Works
--------------
lambda_function.py contains the full function code.
On invocation, it reads the uploaded object's bucket and key from the S3 event.
Downloads the image into memory (no disk writes).
Resizes proportionally if wider than MAX_WIDTH (800px).
Adds a configurable text watermark (WATERMARK_TEXT) to the bottom-right corner.
Saves as JPEG at JPEG_QUALITY (70) to reduce file size.
Uploads the result to the destination bucket (set via the DESTINATION_BUCKET environment variable) with the prefix processed.

Deployment Steps
-----------------
Create two S3 buckets — one for uploads, one for processed output.
Create an IAM role for Lambda with AmazonS3FullAccess and AWSLambdaBasicExecutionRole policies attached.
Create a Lambda function (Python 3.12 runtime) using that IAM role.
Add the Pillow Lambda layer via its Klayers ARN (region and Python-version specific).
Paste in lambda_function.py and deploy.
Set the DESTINATION_BUCKET environment variable to the processed bucket's name.
Increase memory to 256 MB and timeout to 30 seconds.
Add an S3 trigger on the upload bucket for "All object create events."
Upload a test image and confirm the processed version appears in the destination bucket.

Example Result
----------------
Original Image    
<img width="1920" height="660" alt="Screenshot 2026-08-20 170626" src="https://github.com/user-attachments/assets/d87e6f89-63bc-4daa-a7e8-d583886e6c2b" />

Processed image
<img width="1920" height="678" alt="Screenshot 2026-08-20 170637" src="https://github.com/user-attachments/assets/6c45d962-d7ba-4e65-bebb-2caf34460d4a" />


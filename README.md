# CLOUD-LAB2027
Lab 10 — Deploy Static Web Application Using S3 on AWS
Aim
To deploy a static web application using S3 on AWS and secure it with signed URLs.

Requirements
AWS Account
Amazon S3
Amazon CloudFront
HTML
CSS
Web Browser
Step 1: Create S3 Bucket
Login to the AWS Management Console.
Search for S3.
Open Amazon S3.
Click Create bucket.
Select the required AWS Region.
Enter a unique Bucket name.
Under Block Public Access settings, uncheck Block all public access.
Acknowledge the warning.
Keep the remaining settings as default.
Click Create bucket.
Step 2: Enable Static Website Hosting
Open the newly created S3 bucket.

Click the Properties tab.

Scroll down to Static website hosting.

Click Edit.

Select Enable.

Select Host a static website.

Enter the following as the Index document:

index.html

Click Save changes.

Step 3: Create Website Files
Create the following folder structure:

Lab10/
├── index.html
└── style.css
index.html
Create a file named index.html and add:

<!DOCTYPE html>
<html>
<head>
    <title>Cloud Computing Lab 10</title>
    <link rel="stylesheet" href="style.css">
</head>

<body>

    <h1>Cloud Computing Lab 10</h1>

    <h2>Static Web Application</h2>

    <p>
        This website is deployed using Amazon S3
        and AWS CloudFront.
    </p>

    <button>Welcome to AWS</button>

</body>
</html>
style.css
Create a file named style.css and add:

body {
    font-family: Arial, sans-serif;
    text-align: center;
    background-color: lightblue;
    padding-top: 100px;
}

h1 {
    color: darkblue;
}

h2 {
    color: black;
}

p {
    font-size: 20px;
}

button {
    padding: 10px 20px;
    font-size: 16px;
}
Step 4: Add Bucket Policy
Open the S3 bucket.

Click Permissions.

Scroll down to Bucket policy.

Click Edit.

Add the following policy.

Replace BUCKET_NAME with your actual bucket name.

{ "Version": "2012-10-17", "Statement": [ { "Sid": "PublicReadGetObject", "Effect": "Allow", "Principal": "", "Action": "s3:", "Resource": [ "arn:aws:s3:::BUCKET_NAME/*", "arn:aws:s3:::BUCKET_NAME" ] } ] }

Click Save changes.

Step 5: Upload Website Files
Open the S3 bucket.

Click Objects.

Click Upload.

Select the following files:

index.html

style.css

Click Upload.

Verify that both files are displayed in the bucket.

Step 6: Test S3 Website
Go to the Properties tab.
Find Static website hosting.
Copy the Bucket website endpoint.
Open the endpoint in a web browser.
Verify that the website is displayed.
Step 7: Create CloudFront Distribution
Open the AWS Management Console.

Search for CloudFront.

Open CloudFront.

Click Create distribution.

Under Origin domain, use the S3 static website URL.

Remove https:// from the beginning if required.

Set the viewer protocol policy to:

Redirect HTTP to HTTPS

Configure the allowed HTTP methods.

Click Create distribution.

Step 8: Wait for CloudFront Deployment
Open the CloudFront Distributions page.
Find your newly created distribution.
Wait until the distribution status becomes Deployed.
Step 9: Open CloudFront Website
Copy the Distribution domain name.
Open a new browser tab.
Enter the CloudFront URL.
Example:

https://xxxxxxxxxxxx.cloudfront.net
The static website should be displayed.
Step 10: Verify Output
The website should display:

Cloud Computing Lab 10

Static Web Application

This website is deployed using Amazon S3 and AWS CloudFront.

Welcome to AWS

Experiment Flow
AWS Management Console
        ↓
    Amazon S3
        ↓
  Create S3 Bucket
        ↓
Enable Static Website Hosting
        ↓
  Create Website Files
        ↓
   Add Bucket Policy
        ↓
  Upload Website Files
        ↓
   Test S3 Website
        ↓
   Amazon CloudFront
        ↓
  Create Distribution
        ↓
 Wait for Deployment
        ↓
Copy CloudFront Domain
        ↓
   Open Website
Result
The static web application was successfully deployed using Amazon S3 and accessed through Amazon CloudFront.

Conclusion
Amazon S3 was used to store and host the static website files. Static website hosting was enabled, a bucket policy was configured, the website files were uploaded, and Amazon CloudFront was used to distribute the website.
..................................................................................................................................................................
azure-lab
Implementation of Thread-Based Image Processing Application in Microsoft Azure
Step 1: Create Azure Account
Go to Microsoft Azure.
Sign in using an existing Microsoft account or create a new account.
Open the Azure Portal.
Step 2: Create Storage Account
In the Azure Portal, search for Storage Accounts.
Click Create.
Enter the required details:
Resource Group: Create a new resource group.
Storage Account Name: imagestorage123
Region: Select the nearest region, such as Central India.
Click Review + Create.
Click Create.
Note: The storage account name must be globally unique. If the name is unavailable, use another unique name.

Step 3: Create Blob Container
Open the created Storage Account.
Go to Data Storage → Containers.
Click + Container.
Enter the container name:
images

Step 4: Upload Sample Images
Open the images container.
Click Upload.
Select multiple image files from your computer.
Click Upload.
Example input images:

image1.jpg image2.jpg image3.jpg

Step 5: Get Connection String
Open the created Storage Account.
Go to Security + networking → Access keys.
Copy the Connection string.
Keep the connection string securely for use in the Python program.
Important: Do not upload your Azure connection string or access keys to GitHub.

Step 6: Install Required Libraries
Open Command Prompt or Terminal and execute:

pip install azure-storage-blob pillow

Step 7: Write Multithreaded Python Code
Create a Python file named app.py and add the following code:

import threading
from azure.storage.blob import BlobServiceClient
from PIL import Image
import io

# Azure connection
connection_string = "YOUR_CONNECTION_STRING"
container_name = "images"

blob_service_client = BlobServiceClient.from_connection_string(
    connection_string
)


def process_image(blob_name):

    print(f"Processing: {blob_name}")

    # Get blob client
    blob_client = blob_service_client.get_blob_client(
        container=container_name,
        blob=blob_name
    )

    # Download image
    data = blob_client.download_blob().readall()

    # Create image stream
    stream = io.BytesIO(data)

    # Open and process image
    img = Image.open(stream)
    img = img.convert("RGB")
    img = img.resize((200, 200))

    # Save processed image
    output = io.BytesIO()
    img.save(output, format="JPEG")
    output.seek(0)

    # Create new image name
    new_name = "processed_" + blob_name

    # Upload processed image
    blob_service_client.get_blob_client(
        container=container_name,
        blob=new_name
    ).upload_blob(
        output,
        overwrite=True
    )

    print(f"Completed: {blob_name} -> {new_name}")


def main():

    # Get container client
    container_client = blob_service_client.get_container_client(
        container_name
    )

    # List all blobs
    blobs = container_client.list_blobs()

    # Store threads
    threads = []

    # Create and start a thread for each image
    for blob in blobs:

        # Skip already processed images
        if blob.name.startswith("processed_"):
            continue

        thread = threading.Thread(
            target=process_image,
            args=(blob.name,)
        )

        threads.append(thread)
        thread.start()

    # Wait for all threads to complete
    for thread in threads:
        thread.join()

    print("\nAll images processed successfully")


if __name__ == "__main__":
    main()
Step 8: Run the Application
Replace:
connection_string = "YOUR_CONNECTION_STRING"

Step 9: Verify the Output
Go to the Azure Portal.
Open the created Storage Account.
Navigate to Data Storage → Containers.
Open the images container.
Verify that the processed images have been created.
The images container should contain:

image1.jpg image2.jpg image3.jpg processed_image1.jpg processed_image2.jpg processed_image3.jpg

Output
Terminal Output
Processing: image1.jpg Processing: image2.jpg Processing: image3.jpg

Completed: image1.jpg -> processed_image1.jpg Completed: image2.jpg -> processed_image2.jpg Completed: image3.jpg -> processed_image3.jpg

All images processed successfully

Processed Image Details
Property	Details
Input Images	image1.jpg, image2.jpg, image3.jpg
Processing Method	Python Multithreading
Image Processing	Image Resizing
Original Location	Azure Blob Storage
Output Location	Azure Blob Storage
Output Image Size	200 × 200 pixels
Output Format	JPEG
Output Naming	processed_<original_filename>
Number of Threads	One thread per image
Status	Successfully Processed
................................................................................................................................................................
Salesforce Outbound Email via Apex
This repository contains a simple, lightweight Apex script to send a single outbound email from Salesforce using the Messaging.SingleEmailMessage class. It is ideal for testing email deliverability and learning Apex scripting through the Salesforce Developer Console.

💻 Apex Source Code
Copy and paste the following Apex code directly into the Execute Anonymous Window:

// 1. Initialize the email object
Messaging.SingleEmailMessage email = new Messaging.SingleEmailMessage();

// 2. Set the target recipient email address
String[] toAddresses = new String[] {'naik.girish733@gmail.com'};
email.setToAddresses(toAddresses);

// 3. Set the subject and body of the email
email.setSubject('Hello from Salesforce Developer Console!');
email.setPlainTextBody(
    'Success! This email was sent using Apex code in the Developer Console.'
);

// 4. Send the email
Messaging.sendEmail(
    new Messaging.SingleEmailMessage[] { email }
);

// 5. Print a confirmation message to the logs
System.debug('🚀 Email command sent to Salesforce servers!');
Salesforce Email Screenshot

## 🚀 Step-by-Step Execution Guide
Step 1: Open the Developer Console
Log in to your Salesforce account.
Click the Gear Icon (⚙️) in the top-right corner of your Salesforce Lightning homepage.
Select Developer Console.
Step 2: Open the Execute Anonymous Window
In the Developer Console:

Click Debug from the menu bar.

Select Open Execute Anonymous Window.

Alternatively, use:

Windows: Ctrl + E
Mac: Cmd + E
Step 3: Paste and Modify the Code
Clear any existing code from the window.
Paste the Apex code provided above.
Update the recipient email address if required:
String[] toAddresses = new String[] {'your-email@example.com'};
Step 4: Execute the Script
Check Open Log at the bottom-right of the execution window.
Click Execute.
Step 5: Verify the Logs
Once the execution log opens:

Select the Debug Only checkbox at the bottom of the log viewer.
Look for the following debug message:
USER_DEBUG|[18]|DEBUG|🚀 Email command sent to Salesforce servers!
Salesforce Email Screenshot

This confirms that the Apex script successfully submitted the email request to Salesforce.

Step 6: Check Your Inbox
Open the recipient's email inbox and look for the message.

If the email does not appear in the primary inbox within a few minutes, check the Spam/Junk folder.

Note: Successful execution of Messaging.sendEmail() means Salesforce accepted the email request. Actual delivery can still depend on Salesforce email settings, organization limits, recipient mail-server policies, and spam filtering.

Salesforce Email Screenshot

🛠️ Technologies Used
Salesforce
Apex
Messaging.SingleEmailMessage
Salesforce Developer Console
................................................................................................................................................................
<img width="1892" height="962" alt="image" src="https://github.com/user-attachments/assets/f8fc3f9e-690f-4955-a32d-7034d001876d" />

................................................................................................................................................................
<img width="1795" height="876" alt="image" src="https://github.com/user-attachments/assets/a199c1ff-616f-44af-8fdb-8a0ba6856fa1" />

................................................................................................................................................................
<img width="1762" height="586" alt="image" src="https://github.com/user-attachments/assets/c3c35729-bc0e-4d66-8e00-5f70f3643998" />

................................................................................................................................................................
<img width="1723" height="913" alt="image" src="https://github.com/user-attachments/assets/e5fb304d-30ec-4c52-9bee-b1ba44ce6cc5" />

................................................................................................................................................................
<img width="1811" height="868" alt="image" src="https://github.com/user-attachments/assets/1c978870-e02b-4647-b920-90a095ff8859" />

................................................................................................................................................................
Lab 11 — Create Video Streaming Service Using S3 and CloudFront
Aim
To create a video streaming service using S3 and CloudFront and AWS Elemental MediaConvert Digital Rights Management System.

Step 1: Set Up Amazon S3 Bucket
Login to the AWS Management Console.

Go to Services → S3.

Click Create bucket.

Enter a unique bucket name.

Example:

my-video-streaming-bucket-2026

Select the required AWS Region.

Keep Block all public access checked.

Enable Bucket Versioning.

Enable Default Encryption using SSE-S3.

Click Create bucket.

Step 2: Upload a Video
Open the newly created S3 bucket.
Click Upload.
Select a test .mp4 video.
Click Upload.
Check that the video is available in the bucket.
Example:

my-video-streaming-bucket-2026/
└── sample.mp4
Step 3: Create CloudFront Distribution
Search for CloudFront in the AWS Console.
Open CloudFront.
Click Create distribution.
Origin Settings
Under Origin domain, select the S3 bucket.

Under Origin access, select:

Origin access control settings (recommended)

Click Create control setting.

Keep the default settings.

Click Create.

Default Cache Behavior Settings
Under Viewer protocol policy, select:

Redirect HTTP to HTTPS

Under Allowed HTTP methods, select:

GET, HEAD

Under Cache key and origin requests, select:
Cache policy and origin request policy (recommended)

Select:
CachingOptimized

For the basic lab, select:
Do not enable security protections

Click Create distribution.
Step 4: Update S3 Bucket Policy
After creating the CloudFront distribution, click Copy policy.
Go back to S3.
Open your video bucket.
Click Permissions.
Scroll down to Bucket policy.
Click Edit.
Paste the copied CloudFront policy.
Click Save changes.
Example Policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::video-streaming-bucket/*"
    }
  ]
}
Step 5: Test the Streaming Service
Go to the CloudFront Distributions page.
Wait until the distribution is deployed.
Copy the Distribution domain name.
Example:

d111111abcdef8.cloudfront.net
Open a new browser tab.
Enter the CloudFront domain name.
Add the uploaded video file name at the end of the URL.
Example:

https://d111111abcdef8.cloudfront.net/sample.mp4
Press Enter.
The video should load through the CloudFront stream.
Result
The video streaming service was successfully created using Amazon S3 and Amazon CloudFront.

..................................................................................................................................................................
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCEM Student Profile</title>

    <style>
        :root {
            --primary: #00e5ff;
            --secondary: #7c4dff;
            --glass: rgba(255,255,255,0.08);
            --text: #ffffff;
        }

        body {
            margin: 0;
            padding: 50px 20px;
            font-family: "Segoe UI", Arial, sans-serif;
            color: var(--text);

            /* New Background */
            background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
            background-attachment: fixed;

            display: flex;
            justify-content: center;
        }

        .main-wrapper {
            width: 100%;
            max-width: 900px;
        }

        header {
            text-align: center;
            margin-bottom: 40px;
        }

        header h1 {
            font-size: 3rem;
            background: linear-gradient(to right,#00e5ff,#7c4dff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            margin-bottom: 10px;
        }

        .card {
            background: var(--glass);
            backdrop-filter: blur(18px);
            border: 1px solid rgba(255,255,255,0.12);
            border-radius: 20px;
            padding: 30px;
            margin-bottom: 25px;
            transition: 0.3s;
        }

        .card:hover {
            transform: translateY(-8px);
            border-color: var(--primary);
            box-shadow: 0 10px 25px rgba(0,0,0,.3);
        }

        h2 {
            color: var(--primary);
        }

        .badge {
            display: inline-block;
            padding: 8px 18px;
            border-radius: 30px;
            background: var(--secondary);
            margin-bottom: 20px;
            font-weight: bold;
        }

        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fit,minmax(250px,1fr));
            gap: 15px;
        }

        ul {
            list-style: none;
            padding: 0;
        }

        li {
            margin: 12px 0;
            padding-left: 15px;
            border-left: 4px solid var(--primary);
        }

        strong {
            color: #00e5ff;
        }
    </style>
</head>

<body>

<div class="main-wrapper">

<header>
<h1>Sahyadri College of Engineering and Management</h1>
</header>

<div class="card">

<h2>College Vision</h2>

<p>
To be a premier institution in Technology and Management by fostering excellence
in education, innovation, incubation and values to inspire and empower young minds.
</p>

<h2>Mission</h2>

<ul>
<li>Creating an academic ambience to impart holistic education.</li>
<li>Developing skill-based learning through industry interaction.</li>
<li>Fostering innovation through state-of-the-art infrastructure.</li>
</ul>

</div>

<div class="card">

<div class="badge">Student Profile</div>

<div class="grid">

<p><strong>Name:</strong> prasad achari</p>

<p><strong>USN:</strong> 4SF24CS150</p>

<p><strong>Department:</strong> Computer Science & Engineering</p>

<p><strong>Section:</strong> 7C</p>

</div>

</div>

<div class="card">

<h2>Department of Computer Science & Engineering</h2>

<p>
To be a globally recognized center for imparting quality technical education through
innovative research, industry collaboration, and ethical values.
</p>

<h2>Mission</h2>

<ul>
<li>Provide quality technical education with practical exposure.</li>
<li>Encourage innovation, research and entrepreneurship.</li>
<li>Develop ethical professionals and future leaders.</li>
<li>Promote lifelong learning and continuous improvement.</li>
</ul>

</div>

</div>

</body>
</html>


...............................................................................................................................................................
Commands

sudo dnf update -y

sudo dnf install httpd -y

sudo systemctl start httpd

sudo systemctl enable httpd

cd /var/www/html

sudo nano index.html

.................................................................................................................................................................

alesforce Outbound Email via Apex
This repository contains a simple, lightweight Apex script to send a single outbound email from Salesforce using the Messaging.SingleEmailMessage class. It is ideal for testing email deliverability and learning Apex scripting through the Salesforce Developer Console.

💻 Apex Source Code
Copy and paste the following Apex code directly into the Execute Anonymous Window:

// 1. Initialize the email object
Messaging.SingleEmailMessage email = new Messaging.SingleEmailMessage();

// 2. Set the target recipient email address
String[] toAddresses = new String[] {'prasadachari18@gmail.com'};
email.setToAddresses(toAddresses);

// 3. Set the subject and body of the email
email.setSubject('Hello from Salesforce Developer Console!');
email.setPlainTextBody(
    'Success! This email was sent using Apex code in the Developer Console.'
);

// 4. Send the email
Messaging.sendEmail(
    new Messaging.SingleEmailMessage[] { email }
);

// 5. Print a confirmation message to the logs
System.debug('🚀 Email command sent to Salesforce servers!');
Salesforce Email Screenshot

## 🚀 Step-by-Step Execution Guide
Step 1: Open the Developer Console
Log in to your Salesforce account.
Click the Gear Icon (⚙️) in the top-right corner of your Salesforce Lightning homepage.
Select Developer Console.
Step 2: Open the Execute Anonymous Window
In the Developer Console:

Click Debug from the menu bar.

Select Open Execute Anonymous Window.

Alternatively, use:

Windows: Ctrl + E
Mac: Cmd + E
Step 3: Paste and Modify the Code
Clear any existing code from the window.
Paste the Apex code provided above.
Update the recipient email address if required:
String[] toAddresses = new String[] {'your-email@example.com'};
Step 4: Execute the Script
Check Open Log at the bottom-right of the execution window.
Click Execute.
Step 5: Verify the Logs
Once the execution log opens:

Select the Debug Only checkbox at the bottom of the log viewer.
Look for the following debug message:
USER_DEBUG|[18]|DEBUG|🚀 Email command sent to Salesforce servers!
Salesforce Email Screenshot

This confirms that the Apex script successfully submitted the email request to Salesforce.

Step 6: Check Your Inbox
Open the recipient's email inbox and look for the message.

If the email does not appear in the primary inbox within a few minutes, check the Spam/Junk folder.

Note: Successful execution of Messaging.sendEmail() means Salesforce accepted the email request. Actual delivery can still depend on Salesforce email settings, organization limits, recipient mail-server policies, and spam filtering.

Salesforce Email Screenshot

🛠️ Technologies Used
Salesforce
Apex
Messaging.SingleEmailMessage
Salesforce Developer Console

...................................................................................................................................................................

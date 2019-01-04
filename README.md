# AWSElasticsearchBuilding an end to end platform using twitter feed, and sentiment analysis on AWS platform
# Overview
We are living in the age of big data, advanced analytics, and data science. The trend of “big data growth” has not only triggered tremendous hype and buzz, but more importantly presents enormous challenges, which in turn have brought incredible innovation and economic opportunities. (Cao, 2018) therefor our use is case to empower AWS platform with big data technology to analyse twitter stream live for a specific wonderful time this year. Christmas.! Although it was a technological practice for Big Data engineer and data scientist but by a small tweak this method can be applied to almost any situation and all social media platforms.
Christmas is the best time of the year. Have a good one and make the most of it! It was a wonderful Christmas evening that we tried to use public twitter stream to analyse how people use ‘Christmas’ word to celebrate these days and send congratulation to each other. In this practice we used node technology installed on AWS EC2, Amazon kinesis Firehose, AWS Lambda and Amazon Elasticsearch and Kibana services. Although we filter out twitter stream and remove any tweet without location therefore polished data will be ready to apply on a map. Below is the one the generic outputs from this practice: (just reminding that the clock was 4:00AM London time)

 
The work image above shows tweet data on Christmas time when tweet data includes location and twits are included with Christmas word.
•	Tweets fitler out by the word Christmas and visualized
•	Sentiment analysis applied into the data before loading to Kibana
•	Map visualization – whole world selected.

# Architecture
This process has the following architecture to make nearly real-time process using available tools:
•	A container device (using NodeJS to provide, the Twitter feed) and trigger data ingestion into Amazon Kinesis Firehose.
•	Firehose automatically collects and saves data into Amazon S3 default configuration.
•	A Python Lambda function to index S3 file data and provide valuable fields for to Amazon Elasticsearch Service.
•	Visualization tools (Kibana application + Elasticsearch) to visualize indexed data.

 

# Requirements
To create this end to end process, we need an active AWS account and a Twitter active application. For twitter account, we created a  create a new application at https://developer.twitter.com/en/apps  and provide tokens to connect form NodeJS configuration file. There are four pieces of information for tokens which you can see them below: consumer key (API key), consumer secret (API secret), access token, and access token secret. Write down these keys.
 

 

Twitter streaming data using NodeJS
Node.js application connects (source code provided in the link below) to the Twitter feed via the Twitter stream API (from requirement section) and saves the streaming data into Firehose. NodeJS application installed in an AWS EC2 server but can be installed in any other environments as well.
Note: We will provide a full detail document for this section if you have any problem to do it yourself
Open for edit the Config.js file with new settings, change parameters as follows:
•	firehose
o	DeliveryStreamName – Name your desire stream. The app creates the delivery stream if it does not exist.
o	BucketARN: Use the bucket matched to the Lambda function.
o	RoleARN: Get your account ID from the IAM dashboard users sign-in link 
o	Prefix: Use the same s3 prefix that you used in your Lambda function event source (e.g., twitter/raw-data/).
o	twitter – Enter your twitter application keys.
o	region – Enter your Firehose region (e.g., us-east-1, us-west-2, eu-west-1).
o	Make sure your AWS credentials are configured under <HOME FOLDER>/.aws/credentials as follows
For running application, we used this command on EC2:
Node twitter_stream_producer_app
It takes couple of minutes until the data will be delivered to Amazon S3 because firehouse buffers the incoming stream and stores them in the file. pattern of the stored file in S3 will be as below:
/<your bucket>/ <year>/<day>/

Download Application source code from here
Important note: this flow can be transfer and process huge amount of data in a very short period, so if you want to avoid unnecessary cost, while you have couple files in S3 bucket you can stop services and continue from this section with existing files.
# Lambda function
For this flow, we used a Python function (lambda_function.py) that is fired once a file landed in S3 bucket. In high level the function does below:
1.	Reads the file content and convert it to the JSON format
2.	Analyzes Twitter data (tweet_utils.py):
•	Extracts sentiment based on emoticons. If there’s no emoticon in the text the function uses sentiment analysis.
3.	Loads the data to Elasticsearch Service (twitter_to_es.py) using the elasticsearch-py library.
The source code can be downloaded from this location 
Steps for installing Lambda function
1.	Download the deployment package and unzip it in your location folder.
2.	Open and modify the s3-twitter-to-es-python/config.py file by changing the value of es_host to the Elasticsearch Service endpoint of your domain.
3.	Zip the folder content on your local environment as my-s3-twitter-to-es-python.zip (important: zip the folder content, not the folder itself).
4.	Sign in to the Lambda console.
5.	Choose Create a Lambda function (or Get started now if this is your first time using the service).
6.	Choose Skip in the blueprints screen.
7.	Name your function (e.g., s3-twitter-to-es-python or s3-twitter-to-es-python2) .
8.	Choose Python 2.7 runtime and upload the zip file my-s3-twitter-to-es-python.zip.
9.	Make sure the Handler field value is lambda_function.lambda_handler.

 
 
10.	Choose lambda_s3_exec_role (if this value does not exist, choose Create new role S3 execution role).
11.	Keep memory at 128MB and choose a 2min timeout.
12.	Choose Next and Create function, then wait until the function is created.
13.	On the Event sources tab, choose Add event source.
14.	Choose the event source type S3, select the bucket, and choose the event type Object Created (All).
15.	Enter a value for S3 Prefix (e.g., twitter/raw-data/) to ensure the function doesn’t trigger when data is uploaded elsewhere in the bucket.
16.	Make sure that the event source is enabled and click Submit.

# IAM role for Firehose
Use a Firehose delivery stream to ingest the Twitter streaming data and save them it to Amazon S3. Before you can ingest the data into Firehose, you need to set up an IAM role to allow Firehose to call AWS services on your behalf. In this example, the Twitter feed which is your producer application creates the Firehose delivery stream based on the IAM role.
Follow these steps to create IAM role named “firehose_delivery_role” using AWS CLI:
1.	Create the following files:
firehose.json: has been added here 
2.	Make sure the configuration is arranging with your own configuration
3.	Run the following commands:
aws iam create-role --role-name firehose_delivery_role --assume-role-policy-document file://firehose-policy.json
aws iam put-role-policy --role-name firehose_delivery_role --policy-name firehose-s3-rw --policy-document file://s3-rw-policy.json

# Amazon Elasticsearch Service cluster
Start by creating an Amazon Elasticsearch Service cluster that will hold your data for near real-time analysis. Elasticsearch Service includes built-in support for Kibana, which is used for visualization on top of Elasticsearch Service.
1.	Sign in to the Amazon Elasticsearch Service console.
2.	Choose Create a new domain (or Get Started, if this is your first time in the console).
3.	Name your domain “es-twitter-demo” and choose Next.
4.	Keep the default selections and choose Next.
5.	Choose the Allow open access to the domain template for the access policy and click Next.
Note: This is not a recommended approach and should only be used for this demo. Please read the documentation for how to setup the proper permissions.
3.	Choose Confirm and create.
4.	Within ~10 minutes, your domain is ready. When the creation process has reached a status of Active, the domain should be associated with both an Elasticsearch Service endpoint and a Kibana URL, which you need to store for later steps.
5.	Enjoy playing with Elasticsearch and Kibana 

 
 

 

 

 

 

# Conclusion
Using AWS technologies is a journey without a final destination. Focusing on business logic is easier while architects think up freely and combine advanced components to response faster and easier to the changing environment. AWS offers an easy way to tie these services together to create a near real-time discovery analytical big data platform. For this post, we analysed the data from twitter at the end of 2018 Christmas time, but this type of platform can be used for a variety of other use cases.
Feel free to send me your suggestion in case of any interesting business logic.
# What’s Next ?
Explore this architecture with the powerful EMR


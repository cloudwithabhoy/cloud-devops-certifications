# Sample Questions — AWS Certified Solutions Architect (SAA-C03)

## Question 1

A bicycle sharing company is developing a multi-tier architecture to track the location of its bicycles during peak operating hours. The company wants to use these data points in its existing analytics platform. A solutions architect must determine the most viable multi-tier option to support this architecture. The data points must be accessible from the REST API.

Which action meets these requirements for storing and retrieving location data?

- A. Use Amazon Athena with Amazon S3.
- B. Use Amazon API Gateway with AWS Lambda.
- C. Use Amazon QuickSight with Amazon Redshift.
- D. Use Amazon API Gateway with Amazon Kinesis Data Analytics.

**Architecture Diagram:**

```
 +-----------+        +-------------+        +------------------------+        +--------------------+
 | Bicycles  | -----> |  Amazon API | -----> | Amazon Kinesis Data    | -----> | Existing Analytics |
 | (GPS data)|  REST  |   Gateway   |        |     Analytics          |        |     Platform        |
 +-----------+        +-------------+        +------------------------+        +--------------------+
```

**Correct Answer: D**

**Explanation:** API Gateway can expose a REST API that ingests the bicycles' real-time location data directly into a Kinesis Data Analytics stream. Kinesis Data Analytics then processes that streaming data with SQL in real time and feeds it into the existing analytics platform, satisfying both the REST API access requirement and the continuous, real-time nature of the data.

- A. Wrong — Athena queries static data already stored in S3 via SQL; it isn't designed for real-time REST-based ingestion/retrieval of streaming location data.
- B. Wrong — API Gateway with Lambda is a general-purpose serverless REST pattern, but it has no built-in mechanism for real-time stream analytics, so it doesn't fit the "existing analytics platform" requirement as directly as Kinesis Data Analytics.
- C. Wrong — QuickSight with Redshift is a BI/visualization and data warehousing combo, not a REST-accessible mechanism for storing/retrieving live location data.

## Question 2

A solutions architect needs to implement a solution to reduce a company's storage costs. All the company's data is in the Amazon S3 Standard storage class. The company must keep all data for at least 25 years. Data from the most recent 2 years must be highly available and immediately retrievable.

Which solution will meet these requirements?

- A. Set up an S3 Lifecycle policy to transition objects to S3 Glacier Deep Archive immediately.
- B. Set up an S3 Lifecycle policy to transition objects to S3 Glacier Deep Archive after 2 years.
- C. Use S3 Intelligent-Tiering. Activate the archiving option to ensure that data is archived in S3 Glacier Deep Archive.
- D. Set up an S3 Lifecycle policy to transition objects to S3 One Zone-Infrequent Access (S3 One Zone-IA) immediately and to S3 Glacier Deep Archive after 2 years.

**Architecture Diagram:**

```
 +------------------+   0-2 yrs    +------------------------+
 | Amazon S3        | -----------> | S3 Standard            |
 | (new objects)    |              | (highly available,     |
 +------------------+              |  immediately retrieval) |
                                    +------------------------+
                                              |
                                              | after 2 years
                                              v
                                    +------------------------+
                                    | S3 Glacier Deep Archive |
                                    | (kept until 25 years)   |
                                    +------------------------+
```

**Correct Answer: B**

**Explanation:** An S3 Lifecycle policy that keeps objects in S3 Standard for the first 2 years satisfies the "highly available and immediately retrievable" requirement, then transitions them to S3 Glacier Deep Archive — the lowest-cost storage class — for the remaining ~23 years, satisfying the 25-year retention requirement at minimal cost.

- A. Wrong — Transitioning immediately puts recent data straight into Glacier Deep Archive, which has retrieval times measured in hours, violating the "immediately retrievable" requirement for the most recent 2 years.
- C. Wrong — S3 Intelligent-Tiering's archive tiers can move objects to Glacier-like tiers in as little as 90 days, so data younger than 2 years could lose immediate retrievability, violating the hard 2-year requirement.
- D. Wrong — S3 One Zone-IA immediately reduces availability (single AZ) for the most recent data, which contradicts the "highly available" requirement for the first 2 years.

## Question 3

A company operates a food delivery service. Because of recent growth, the company's order processing system is experiencing scaling problems during peak traffic hours. The current architecture includes Amazon EC2 instances in an Auto Scaling group that collect orders from an application. A second group of EC2 instances in an Auto Scaling group fulfills the orders.

The order collection process occurs quickly, but the order fulfillment process can take longer. Data must not be lost because of a scaling event.

A solutions architect must ensure that the order collection process and the order fulfillment process can both scale adequately during peak traffic hours.

Which solution will meet these requirements?

- A. Use Amazon CloudWatch to monitor the CPUUtilization metric for each instance in both Auto Scaling groups. Configure each Auto Scaling group's minimum capacity to meet its peak workload value.
- B. Use Amazon CloudWatch to monitor the CPUUtilization metric for each instance in both Auto Scaling groups. Configure a CloudWatch alarm to invoke an Amazon Simple Notification Service (Amazon SNS) topic to create additional Auto Scaling groups on demand.
- C. Provision two Amazon Simple Queue Service (Amazon SQS) queues. Use one SQS queue for order collection. Use the second SQS queue for order fulfillment. Configure the EC2 instances to poll their respective queues. Scale the Auto Scaling groups based on notifications that the queues send.
- D. Provision two Amazon Simple Queue Service (Amazon SQS) queues. Use one SQS queue for order collection. Use the second SQS queue for order fulfillment. Configure the EC2 instances to poll their respective queues. Scale the Auto Scaling groups based on the number of messages in each queue.

**Architecture Diagram:**

```
 +-------------+     +----------------------+     +----------------------------+
 | Application | --> | SQS: Order Collection| --> | ASG: Collection Instances  |
 +-------------+     +----------------------+     +----------------------------+
                                                              |
                                                              v
                                          +----------------------------+
                                          | SQS: Order Fulfillment    |
                                          +----------------------------+
                                                              |
                                                              v
                                          +----------------------------+
                                          | ASG: Fulfillment Instances |
                                          | (scales on queue depth)   |
                                          +----------------------------+
```

**Correct Answer: D**

**Explanation:** Two decoupled SQS queues let orders sit safely in the queue if instances are scaling, so no data is lost. Scaling each Auto Scaling group based on the number of messages in its queue (queue depth / backlog per instance) directly reflects the actual workload each stage needs to process, letting collection and fulfillment scale independently and adequately, even though fulfillment takes longer.

- A. Wrong — CPUUtilization doesn't reflect the actual backlog of unprocessed orders; an idle-looking CPU could still have a large queue backlog, so this metric under-scales during traffic spikes.
- B. Wrong — Creating additional Auto Scaling groups on demand is not how ASGs work and adds unnecessary complexity; it doesn't address the core scaling metric problem.
- C. Wrong — SQS queues don't natively send "scaling notifications"; the standard and correct approach is to scale on a CloudWatch metric (queue message count), not on notifications from the queue.

## Question 4 (Choose two)

A company produces batch data that comes from different databases. The company also produces live stream data from network sensors and application APIs. The company needs to consolidate all the data into one place for business analytics. The company needs to process the incoming data and then stage the data in different Amazon S3 buckets. Teams will later run one-time queries and import the data into a business intelligence tool to show key performance indicators (KPIs).

Which combination of steps will meet these requirements with the LEAST operational overhead? (Choose two.)

- A. Use Amazon Athena for one-time queries. Use Amazon QuickSight to create dashboards for KPIs.
- B. Use Amazon Kinesis Data Analytics for one-time queries. Use Amazon QuickSight to create dashboards for KPIs.
- C. Create custom AWS Lambda functions to move the individual records from the databases to an Amazon Redshift cluster.
- D. Use an AWS Glue extract, transform, and load (ETL) job to convert the data into JSON format. Load the data into multiple Amazon OpenSearch Service (Amazon Elasticsearch Service) clusters.
- E. Use blueprints in AWS Lake Formation to identify the data that can be ingested into a data lake. Use AWS Glue to crawl the source, extract the data, and load the data into Amazon S3 in Apache Parquet format.

**Architecture Diagram:**

```
 +------------------+     +--------------------+     +------------------------+
 | Databases /      | --> | AWS Lake Formation  | --> | Amazon S3 Data Lake   |
 | Sensors / APIs   |     | + AWS Glue (crawl,  |     |    (Apache Parquet)   |
 +------------------+     |    extract, load)   |     +------------------------+
                          +--------------------+                 |
                                                                  v
                                                       +------------------------+
                                                       | Amazon Athena          |
                                                       | (one-time queries)     |
                                                       +------------------------+
                                                                  |
                                                                  v
                                                       +------------------------+
                                                       | Amazon QuickSight      |
                                                       |  (KPI dashboards)      |
                                                       +------------------------+
```

**Correct Answers: A and E**

**Explanation:** AWS Lake Formation with Glue crawlers (E) automates identifying, extracting, and loading disparate batch and streaming sources into an S3 data lake in the efficient Parquet format, with minimal setup or code. Athena (A) then runs serverless one-time SQL queries directly against that S3 data with no infrastructure to manage, and QuickSight builds the KPI dashboards on top — together this is the lowest-operational-overhead, fully managed/serverless combination.

- B. Wrong — Kinesis Data Analytics is built for continuous streaming SQL analytics, not ad-hoc one-time queries; using it for one-time queries adds unnecessary complexity and cost.
- C. Wrong — Custom Lambda functions to move individual records is a manual, code-heavy approach with high operational overhead compared to a managed Glue/Lake Formation pipeline.
- D. Wrong — Converting all data to JSON and loading into multiple OpenSearch clusters is not aligned with the stated need for S3-staged data and one-time SQL querying, and running multiple search clusters adds significant operational overhead.

## Question 5

A company performs monthly maintenance on its AWS infrastructure. During these maintenance activities, the company needs to rotate the credentials for its Amazon RDS for MySQL databases across multiple AWS Regions.

Which solution will meet these requirements with the LEAST operational overhead?

- A. Store the credentials as secrets in AWS Secrets Manager. Use multi-Region secret replication for the required Regions. Configure Secrets Manager to rotate the secrets on a schedule.
- B. Store the credentials as secrets in AWS Systems Manager by creating a secure string parameter. Use multi-Region secret replication for the required Regions. Configure Systems Manager to rotate the secrets on a schedule.
- C. Store the credentials in an Amazon S3 bucket that has server-side encryption (SSE) enabled. Use Amazon EventBridge (Amazon CloudWatch Events) to invoke an AWS Lambda function to rotate the credentials.
- D. Encrypt the credentials as secrets by using AWS Key Management Service (AWS KMS) multi-Region customer managed keys. Store the secrets in an Amazon DynamoDB global table. Use an AWS Lambda function to retrieve the secrets from DynamoDB. Use the RDS API to rotate the secrets.

**Architecture Diagram:**

```
 +------------------------+     +--------------------------+     +------------------------+
 | AWS Secrets Manager    | --> | Multi-Region Replication | --> | RDS for MySQL          |
 | (stores DB credentials)|     |  (replica secrets)       |     | (Region A, B, C, ...)  |
 +------------------------+     +--------------------------+     +------------------------+
              |
              | scheduled rotation
              v
 +------------------------+
 | Built-in rotation      |
 | (native RDS support)   |
 +------------------------+
```

**Correct Answer: A**

**Explanation:** AWS Secrets Manager has native, built-in support for both multi-Region secret replication and scheduled automatic rotation of RDS credentials (using an AWS-managed Lambda rotation function), requiring no custom code — the lowest operational overhead of the options given.

- B. Wrong — Systems Manager Parameter Store SecureString parameters have no built-in rotation scheduling or multi-Region replication feature; achieving this would require custom automation, adding overhead.
- C. Wrong — Storing credentials in S3 and writing a custom Lambda function to rotate them is a manual, custom-built solution with much higher operational overhead than a managed Secrets Manager rotation.
- D. Wrong — Using KMS, DynamoDB global tables, and a custom Lambda function to retrieve secrets and call the RDS API is a complex custom-built pipeline, far more operational overhead than Secrets Manager's native capabilities.

## Question 6 (Choose two)

A company hosts an application in a private subnet. The company has already integrated the application with Amazon Cognito. The company uses an Amazon Cognito user pool to authenticate users.

The company needs to modify the application so the application can securely store user documents in an Amazon S3 bucket.

Which combination of steps will securely integrate Amazon S3 with the application? (Choose two.)

- A. Create an Amazon Cognito identity pool to generate secure Amazon S3 access tokens for users when they successfully log in.
- B. Use the existing Amazon Cognito user pool to generate Amazon S3 access tokens for users when they successfully log in.
- C. Create an Amazon S3 VPC endpoint in the same VPC where the company hosts the application.
- D. Create a NAT gateway in the VPC where the company hosts the application. Assign a policy to the S3 bucket to deny any request that is not initiated from Amazon Cognito.
- E. Attach a policy to the S3 bucket that allows access only from the users' IP addresses.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | Application      | --> | Cognito User Pool   | --> | Cognito Identity Pool |
 | (private subnet) |     | (authentication)     |     | (temp AWS credentials)|
 +------------------+     +----------------------+     +------------------------+
          |                                                        |
          | S3 VPC Endpoint                                        | scoped IAM role
          v                                                        v
 +------------------------------------------------------------------------+
 |                        Amazon S3 (user documents)                     |
 +------------------------------------------------------------------------+
```

**Correct Answers: A and C**

**Explanation:** A Cognito user pool only authenticates users (verifies identity) — it doesn't grant AWS resource permissions. A Cognito identity pool (A) exchanges the user pool's authentication token for temporary, scoped AWS credentials (via IAM roles) that can securely access S3. Since the application runs in a private subnet with no internet access, an S3 VPC endpoint (C) lets it reach S3 privately without a NAT gateway or public internet route.

- B. Wrong — A user pool handles authentication only; it has no capability to generate AWS service (S3) access tokens — that's the identity pool's role.
- D. Wrong — A NAT gateway routes traffic to the public internet, which isn't required (or ideal) for private S3 access, and S3 bucket policies can't natively "deny requests not initiated from Cognito" — Cognito isn't a request-origin construct.
- E. Wrong — Restricting the bucket policy to users' IP addresses doesn't work reliably for authenticated app users behind NAT/proxies and doesn't provide identity-based access control tied to Cognito authentication.

## Question 7

A company has an automobile sales website that stores its listings in a database on Amazon RDS. When an automobile is sold, the listing needs to be removed from the website and the data must be sent to multiple target systems.

Which design should a solutions architect recommend?

- A. Create an AWS Lambda function triggered when the database on Amazon RDS is updated to send the information to an Amazon Simple Queue Service (Amazon SQS) queue for the targets to consume.
- B. Create an AWS Lambda function triggered when the database on Amazon RDS is updated to send the information to an Amazon Simple Queue Service (Amazon SQS) FIFO queue for the targets to consume.
- C. Subscribe to an RDS event notification and send an Amazon Simple Queue Service (Amazon SQS) queue fanned out to multiple Amazon Simple Notification Service (Amazon SNS) topics. Use AWS Lambda functions to update the targets.
- D. Subscribe to an RDS event notification and send an Amazon Simple Notification Service (Amazon SNS) topic fanned out to multiple Amazon Simple Queue Service (Amazon SQS) queues. Use AWS Lambda functions to update the targets.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | Amazon RDS       | --> | Amazon SNS Topic     | --> | SQS Queue (Target 1)  | --> Lambda --> Target 1
 | (event notify)   |     | (fan-out)            |     +------------------------+
 +------------------+     +----------------------+     +------------------------+
                                    |                    | SQS Queue (Target 2)  | --> Lambda --> Target 2
                                    +------------------> +------------------------+
                                    |                    | SQS Queue (Target N)  | --> Lambda --> Target N
                                    +------------------> +------------------------+
```

**Correct Answer: D**

**Explanation:** SNS is a pub/sub fan-out service designed to broadcast a single message to many subscribers — here, multiple SQS queues, one per target system. Each target then consumes its own SQS queue independently and reliably (with retry/durability), and Lambda functions process each queue to update the respective target. This is the standard SNS-fan-out-to-SQS pattern for one-to-many reliable delivery.

- A. Wrong — A single SQS queue delivers each message to only one consumer; it can't fan a single event out to multiple independent target systems.
- B. Wrong — Same fan-out limitation as A; a FIFO queue adds ordering/dedup guarantees but still doesn't support broadcasting to multiple independent targets.
- C. Wrong — This reverses the correct fan-out direction: SQS cannot fan out to multiple SNS topics (a single SQS message is consumed once), so this doesn't achieve the required one-to-many delivery.

## Question 8

A company has a small Python application that processes JSON documents and outputs the results to an on-premises SQL database. The application runs thousands of times each day. The company wants to move the application to the AWS Cloud. The company needs a highly available solution that maximizes scalability and minimizes operational overhead.

Which solution will meet these requirements?

- A. Place the JSON documents in an Amazon S3 bucket. Run the Python code on multiple Amazon EC2 instances to process the documents. Store the results in an Amazon Aurora DB cluster.
- B. Place the JSON documents in an Amazon S3 bucket. Create an AWS Lambda function that runs the Python code to process the documents as they arrive in the S3 bucket. Store the results in an Amazon Aurora DB cluster.
- C. Place the JSON documents in an Amazon Elastic Block Store (Amazon EBS) volume. Use the EBS Multi-Attach feature to attach the volume to multiple Amazon EC2 instances. Run the Python code on the EC2 instances to process the documents. Store the results on an Amazon RDS DB instance.
- D. Place the JSON documents in an Amazon Simple Queue Service (Amazon SQS) queue as messages. Deploy the Python code as a container on an Amazon Elastic Container Service (Amazon ECS) cluster that is configured with the Amazon EC2 launch type. Use the container to process the SQS messages. Store the results on an Amazon RDS DB instance.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | JSON documents   | --> | Amazon S3 bucket      | --> | AWS Lambda            |
 | (uploaded)       |     | (event trigger)       |     | (runs Python code)    |
 +------------------+     +----------------------+     +------------------------+
                                                                    |
                                                                    v
                                                       +------------------------+
                                                       | Amazon Aurora DB       |
                                                       |       Cluster          |
                                                       +------------------------+
```

**Correct Answer: B**

**Explanation:** S3 event notifications triggering a Lambda function is a fully serverless, event-driven design — Lambda scales automatically with the number of incoming documents, requires no servers to manage, and Aurora provides a highly available, managed relational database. This combination directly satisfies "highly available, maximizes scalability, minimizes operational overhead."

- A. Wrong — Running Python on multiple self-managed EC2 instances requires provisioning, patching, and scaling the fleet manually, which adds operational overhead compared to Lambda.
- C. Wrong — EBS Multi-Attach is designed for specific clustered filesystem/application use cases, not general parallel processing, and still requires managing EC2 instances directly — more overhead than a serverless approach.
- D. Wrong — Running an ECS cluster on the EC2 launch type still requires managing underlying EC2 instances (capacity, scaling, patching), which is more operational overhead than the fully serverless S3 + Lambda design.

## Question 9

A company has an AWS Glue extract, transform, and load (ETL) job that runs every day at the same time. The job processes XML data that is in an Amazon S3 bucket. New data is added to the S3 bucket every day. A solutions architect notices that AWS Glue is processing all the data during each run.

What should the solutions architect do to prevent AWS Glue from reprocessing old data?

- A. Edit the job to use job bookmarks.
- B. Edit the job to delete data after the data is processed.
- C. Edit the job by setting the NumberOfWorkers field to 1.
- D. Use a FindMatches machine learning (ML) transform.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | Amazon S3 bucket | --> | AWS Glue ETL Job     | --> | Processed Output       |
 | (old + new XML)  |     | + Job Bookmarks      |     +------------------------+
 +------------------+     | (tracks processed    |
                          |  state between runs) |
                          +----------------------+
```

**Correct Answer: A**

**Explanation:** AWS Glue job bookmarks persist state between job runs, tracking which data (files, objects, or partitions) has already been processed. On subsequent runs, Glue uses this bookmark to process only new data, preventing costly and redundant reprocessing of old data — with no need to delete source data or change infrastructure.

- B. Wrong — Deleting processed data changes the source data and could break other consumers or auditability requirements; it's a destructive workaround, not the intended mechanism.
- C. Wrong — NumberOfWorkers controls the compute capacity/parallelism of the job, not which data gets processed; it has no effect on preventing reprocessing.
- D. Wrong — FindMatches is an ML transform for deduplicating and matching similar records within data, not a mechanism for tracking already-processed files between job runs.

## Question 10

A company has created an image analysis application in which users can upload photos and add photo frames to their images. The users upload images and metadata to indicate which photo frames they want to add to their images. The application uses a single Amazon EC2 instance and Amazon DynamoDB to store the metadata.

The application is becoming more popular, and the number of users is increasing. The company expects the number of concurrent users to vary significantly depending on the time of day and day of week. The company must ensure that the application can scale to meet the needs of the growing user base.

Which solution meets these requirements?

- A. Use AWS Lambda to process the photos. Store the photos and metadata in DynamoDB.
- B. Use Amazon Kinesis Data Firehose to process the photos and to store the photos and metadata.
- C. Use AWS Lambda to process the photos. Store the photos in Amazon S3. Retain DynamoDB to store the metadata.
- D. Increase the number of EC2 instances to three. Use Provisioned IOPS SSD (io2) Amazon Elastic Block Store (Amazon EBS) volumes to store the photos and metadata.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | Users            | --> | AWS Lambda            | --> | Amazon S3              |
 | (upload photo +  |     | (processes photos,    |     | (stores photos)        |
 |  metadata)       |     |  scales automatically)|     +------------------------+
 +------------------+     +----------------------+
                                     |
                                     v
                          +----------------------+
                          | Amazon DynamoDB      |
                          | (stores metadata)    |
                          +----------------------+
```

**Correct Answer: C**

**Explanation:** Lambda automatically scales out to handle unpredictable, widely varying concurrent load with no capacity planning, replacing the single EC2 instance bottleneck. S3 is the right store for binary photo objects (durable, virtually unlimited, scalable), while DynamoDB continues to handle the metadata it's already suited for — together this fully decouples compute and storage so each scales independently.

- A. Wrong — DynamoDB is a NoSQL database meant for structured metadata/items, not designed or cost-effective for storing large binary image files.
- B. Wrong — Kinesis Data Firehose is a streaming data delivery service for ingesting and loading streaming data into stores like S3/Redshift; it isn't designed to "process photos" (image transformations) or serve as photo storage.
- D. Wrong — Manually adding fixed EC2 instances doesn't handle significant, unpredictable variation in concurrent users well, and EBS volumes are attached to single instances, not a scalable shared store for growing photo storage.

## Question 11

A solutions architect must design a solution that uses Amazon CloudFront with an Amazon S3 origin to store a static website. The company's security policy requires that all website traffic be inspected by AWS WAF.

How should the solutions architect comply with these requirements?

- A. Configure an S3 bucket policy to accept requests coming from the AWS WAF Amazon Resource Name (ARN) only.
- B. Configure Amazon CloudFront to forward all incoming requests to AWS WAF before requesting content from the S3 origin.
- C. Configure a security group that allows Amazon CloudFront IP addresses to access Amazon S3 only. Associate AWS WAF to CloudFront.
- D. Configure Amazon CloudFront and Amazon S3 to use an origin access identity (OAI) to restrict access to the S3 bucket. Enable AWS WAF on the distribution.

**Architecture Diagram:**

```
             +----------------------------------------+
             |     Amazon CloudFront Distribution     |
 +----------+ |  +----------------------------------+  |     +------------------+
 | Users    | --> |   AWS WAF (attached, inspects   | --> | Amazon S3 Origin |
 |          | |  |   every request at the edge)    |  |     | (via OAI only)   |
 +----------+ |  +----------------------------------+  |     +------------------+
             +----------------------------------------+
```

**Correct Answer: D**

**Explanation:** AWS WAF attaches directly to a CloudFront distribution and automatically inspects all incoming requests before they reach the origin — no manual traffic forwarding needed. An origin access identity (OAI) ensures S3 only accepts requests from CloudFront, blocking users from bypassing CloudFront (and WAF) by hitting the S3 URL directly. Together these fully satisfy "all traffic inspected by WAF."

- A. Wrong — WAF doesn't have an ARN that originates requests to S3; S3 bucket policies can't be scoped to "accept requests from WAF" since WAF isn't the entity making the request to S3.
- B. Wrong — CloudFront doesn't "forward requests to WAF" as a separate hop; WAF is associated with and evaluates requests inline as part of the CloudFront distribution itself.
- C. Wrong — S3 doesn't support security groups (those apply to VPC resources like EC2/ENIs, not S3 buckets), so this option describes a mechanism that doesn't exist for S3.

## Question 12

A company's image-hosting website gives users around the world the ability to upload, view, and download images from their mobile devices. The company currently hosts the static website in an Amazon S3 bucket.

Because of the website's growing popularity, the website's performance has decreased. Users have reported latency issues when they upload and download images.

The company must improve the performance of the website.

Which solution will meet these requirements with the LEAST implementation effort?

- A. Configure an Amazon CloudFront distribution for the S3 bucket to improve the download performance. Enable S3 Transfer Acceleration to improve the upload performance.
- B. Configure Amazon EC2 instances of the right sizes in multiple AWS Regions. Migrate the application to the EC2 instances. Use an Application Load Balancer to distribute the website traffic equally among the EC2 instances. Configure AWS Global Accelerator to address global demand with low latency.
- C. Configure an Amazon CloudFront distribution that uses the S3 bucket as an origin to improve the download performance. Configure the application to use CloudFront to upload images to improve the upload performance. Create S3 buckets in multiple AWS Regions. Configure replication rules for the buckets to replicate users' data based on the users' location. Redirect downloads to the S3 bucket that is closest to each user's location.
- D. Configure AWS Global Accelerator for the S3 bucket to improve network performance. Create an endpoint for the application to use Global Accelerator instead of the S3 bucket.

**Architecture Diagram:**

```
                        downloads
 +----------+  <----------------------------  +------------------------+
 | Users    |                                 | Amazon CloudFront      |
 | (global) |                                 | Distribution           |
 +----------+  ---------------------------->  +------------------------+
                    uploads (accelerated)                 |
                        |                                  v
                        v                        +------------------------+
              +------------------------+         | Amazon S3 Bucket       |
              | S3 Transfer            | ------> | (origin)               |
              | Acceleration endpoint  |         +------------------------+
              +------------------------+
```

**Correct Answer: A**

**Explanation:** CloudFront caches and serves content from edge locations close to users, directly improving download latency with minimal setup (just create a distribution in front of the existing bucket). S3 Transfer Acceleration uses CloudFront's edge network to speed up uploads to S3 over optimized network paths, again with a simple configuration toggle — no application, infrastructure, or replication changes required, making this the lowest-effort solution.

- B. Wrong — Migrating the entire static website to self-managed EC2 instances across multiple Regions, plus load balancers and Global Accelerator, is a major architectural overhaul — far more implementation effort than enabling two existing S3/CloudFront features.
- C. Wrong — Setting up cross-Region replication, multiple buckets, and location-based upload/download routing logic is significantly more complex and effort-intensive than options A's simple feature toggles.
- D. Wrong — AWS Global Accelerator works with Regional resources such as ALBs, NLBs, and EC2/Elastic IPs — it does not support S3 buckets as endpoints, so this solution is not technically valid.

## Question 13

A global company is using Amazon API Gateway to design REST APIs for its loyalty club users in the us-east-1 Region and the ap-southeast-2 Region. A solutions architect must design a solution to protect these API Gateway managed REST APIs across multiple accounts from SQL injection and cross-site scripting attacks.

Which solution will meet these requirements with the LEAST amount of administrative effort?

- A. Set up AWS WAF in both Regions. Associate Regional web ACLs with an API stage.
- B. Set up AWS Firewall Manager in both Regions. Centrally configure AWS WAF rules.
- C. Set up AWS Shield in both Regions. Associate Regional web ACLs with an API stage.
- D. Set up AWS Shield in one of the Regions. Associate Regional web ACLs with an API stage.

**Architecture Diagram:**

```
 +--------------------------------------------------------+
 |            AWS Firewall Manager (central policy)       |
 |         (centrally configures AWS WAF rules)           |
 +--------------------------------------------------------+
         |                                    |
         v                                    v
 +----------------------+           +----------------------+
 | AWS WAF Web ACL      |           | AWS WAF Web ACL      |
 | (us-east-1)          |           | (ap-southeast-2)     |
 +----------------------+           +----------------------+
         |                                    |
         v                                    v
 +----------------------+           +----------------------+
 | API Gateway (Account | ...       | API Gateway (Account |
 |   A, B, C ...)       |           |   A, B, C ...)       |
 +----------------------+           +----------------------+
```

**Correct Answer: B**

**Explanation:** AWS WAF is the correct service to protect against SQL injection and XSS (via its managed rule groups), but the requirement spans multiple accounts and multiple Regions. AWS Firewall Manager centrally defines and deploys WAF rules across many accounts and Regions from one place, avoiding the need to manually configure WAF web ACLs in each account/Region individually — the least administrative effort for this multi-account scope.

- A. Wrong — Manually setting up WAF web ACLs in both Regions works for a single account, but doing this individually across multiple accounts requires significant repeated manual effort, unlike Firewall Manager's central management.
- C. Wrong — AWS Shield protects against DDoS attacks, not SQL injection or cross-site scripting; it's the wrong service for this requirement regardless of Region count.
- D. Wrong — Same issue as C (Shield doesn't address SQLi/XSS), and additionally only covers one Region when both Regions need protection.

## Question 14

A solutions architect is optimizing a website for an upcoming musical event. Videos of the performances will be streamed in real time and then will be available on demand. The event is expected to attract a global online audience.

Which service will improve the performance of both the real-time and on-demand streaming?

- A. Amazon CloudFront
- B. AWS Global Accelerator
- C. Amazon Route 53
- D. Amazon S3 Transfer Acceleration

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | Live video feed  | --> | Amazon CloudFront     | --> | Global audience       |
 | (real-time)      |     | (edge caching for     |     | (low-latency viewing) |
 +------------------+     |  both live + on-demand)|
 | Video-on-demand  | --> |                       |
 | (Amazon S3 store)|     |                       |
 +------------------+     +----------------------+
```

**Correct Answer: A**

**Explanation:** CloudFront is a content delivery network built to cache and deliver both live (real-time) streaming video and video-on-demand content from edge locations close to viewers worldwide, reducing latency and buffering for a global audience regardless of stream type.

- B. Wrong — AWS Global Accelerator improves performance for TCP/UDP application traffic (via Anycast IPs to Regional endpoints like ALBs/NLBs) but is not a video-optimized CDN with edge caching for streaming content.
- C. Wrong — Route 53 is a DNS service that routes users to endpoints; it doesn't cache or accelerate video content delivery itself.
- D. Wrong — S3 Transfer Acceleration speeds up uploads to S3 buckets, not the delivery/streaming of video content to end viewers.

## Question 15 (Choose two)

A solutions architect needs to help a company optimize the cost of running an application on AWS. The application will use Amazon EC2 instances, AWS Fargate, and AWS Lambda for compute within the architecture.

The EC2 instances will run the data ingestion layer of the application. EC2 usage will be sporadic and unpredictable. Workloads that run on EC2 instances can be interrupted at any time. The application front end will run on Fargate, and Lambda will serve the API layer. The front-end utilization and API layer utilization will be predictable over the course of the next year.

Which combination of purchasing options will provide the MOST cost-effective solution for hosting this application? (Choose two.)

- A. Use Spot Instances for the data ingestion layer
- B. Use On-Demand Instances for the data ingestion layer
- C. Purchase a 1-year Compute Savings Plan for the front end and API layer.
- D. Purchase 1-year All Upfront Reserved Instances for the data ingestion layer.
- E. Purchase a 1-year EC2 Instance Savings Plan for the front end and API layer.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------------+
 | EC2 Spot Instances     |     | AWS Fargate (front end)         |
 | (data ingestion,       |     | AWS Lambda (API layer)          |
 |  interruptible,        |     |                                   |
 |  sporadic workload)    |     | covered by: Compute Savings Plan |
 +------------------------+     | (predictable utilization)        |
                                +----------------------------------+
```

**Correct Answers: A and C**

**Explanation:** Spot Instances offer the deepest discount for workloads that are sporadic and can tolerate interruption — an exact fit for the data ingestion layer. A Compute Savings Plan applies across EC2, Fargate, and Lambda usage, so it can cover the predictable, steady-state front end (Fargate) and API (Lambda) layers at a discount, unlike EC2-specific commitment options.

- B. Wrong — On-Demand pricing has no discount; it's not the most cost-effective option when the workload is sporadic and interruption-tolerant (Spot fits better and is cheaper).
- D. Wrong — Reserved Instances require a steady, predictable commitment, which doesn't match the "sporadic and unpredictable" EC2 data ingestion usage — Reserved Instances also can't apply here since they're EC2-only and this layer wasn't described as steady.
- E. Wrong — An EC2 Instance Savings Plan applies only to EC2 usage; it does not cover Fargate or Lambda usage, so it can't discount the front end and API layer described here — a Compute Savings Plan is needed instead.

## Question 16

A company has a data ingestion workflow that includes the following components:

- An Amazon Simple Notification Service (Amazon SNS) topic that receives notifications about new data deliveries
- An AWS Lambda function that processes and stores the data

The ingestion workflow occasionally fails because of network connectivity issues. When failure occurs, the corresponding data is not ingested unless the company manually reruns the job.

What should a solutions architect do to ensure that all notifications are eventually processed?

- A. Configure the Lambda function for deployment across multiple Availability Zones.
- B. Modify the Lambda function's configuration to increase the CPU and memory allocations for the function.
- C. Configure the SNS topic's retry strategy to increase both the number of retries and the wait time between retries.
- D. Configure an Amazon Simple Queue Service (Amazon SQS) queue as the on-failure destination. Modify the Lambda function to process messages in the queue.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | Amazon SNS Topic | --> | AWS Lambda Function  | -x->| (network failure)      |
 | (new data event) |     | (processes & stores) |     +------------------------+
 +------------------+     +----------------------+
                                     |
                                     | on-failure destination
                                     v
                          +----------------------+
                          | Amazon SQS Queue     |
                          | (failed events kept  |
                          |  for reprocessing)   |
                          +----------------------+
```

**Correct Answer: D**

**Explanation:** Configuring an SQS queue as the Lambda function's on-failure destination captures events that fail processing (e.g., due to transient network issues) instead of discarding them. The data persists safely in the queue and the Lambda function (or another consumer) can process it once connectivity is restored, ensuring no notification is permanently lost without manual intervention.

- A. Wrong — Multi-AZ deployment addresses infrastructure availability, not the handling/retry of individual failed invocations caused by transient network errors.
- B. Wrong — Increasing CPU/memory addresses performance/resource constraints, not network connectivity failures or the loss of failed event data.
- C. Wrong — SNS retries apply to delivering the notification to the subscriber, but once retries are exhausted (or if the failure is on the Lambda processing side), the message is still dropped without a persistent destination like an on-failure queue to catch it.

## Question 17 (Choose three)

A company runs multiple workloads on virtual machines (VMs) in an on-premises data center. The company is expanding rapidly. The on-premises data center is not able to scale fast enough to meet business needs. The company wants to migrate the workloads to AWS.

The migration is time sensitive. The company wants to use a lift-and-shift strategy for non-critical workloads.

Which combination of steps will meet these requirements? (Choose three.)

- A. Use the AWS Schema Conversion Tool (AWS SCT) to collect data about the VMs.
- B. Use AWS Application Migration Service. Install the AWS Replication Agent on the VMs.
- C. Complete the initial replication of the VMs. Launch test instances to perform acceptance tests on the VMs.
- D. Stop all operations on the VMs. Launch a cutover instance.
- E. Use AWS App2Container (A2C) to collect data about the VMs.
- F. Use AWS Database Migration Service (AWS DMS) to migrate the VMs.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+     +------------------+
 | On-premises VMs  | --> | AWS Replication Agent | --> | Initial replication + | --> | Cutover instance |
 | (non-critical    |     | (AWS Application      |     | test/acceptance       |     | (stop VM ops,    |
 |  workloads)      |     |  Migration Service)    |     | instances             |     |  go live on AWS) |
 +------------------+     +----------------------+     +------------------------+     +------------------+
```

**Correct Answers: B, C, and D**

**Explanation:** This is the standard AWS Application Migration Service (MGN) lift-and-shift workflow: (B) install the AWS Replication Agent on each source VM to begin continuous block-level replication into AWS, (C) once initial replication completes, launch non-disruptive test instances to validate the migrated workload, and (D) perform cutover — stop operations on the source VMs and launch the final cutover instance in AWS. This is fast and requires no re-architecting, matching the time-sensitive lift-and-shift requirement.

- A. Wrong — AWS SCT converts database schemas from one engine to another (e.g., Oracle to Aurora); it's unrelated to lift-and-shift of general VM workloads.
- E. Wrong — AWS App2Container is used to containerize existing applications for ECS/EKS, which is a re-platforming strategy, not a lift-and-shift VM migration.
- F. Wrong — AWS DMS migrates databases specifically, not entire VMs; it doesn't fit a general lift-and-shift of on-premises virtual machines.

## Question 18

A security team wants to limit access to specific services or actions in all of the team's AWS accounts. All accounts belong to a large organization in AWS Organizations. The solution must be scalable and there must be a single point where permissions can be maintained.

What should a solutions architect do to accomplish this?

- A. Create an ACL to provide access to the services or actions.
- B. Create a security group to allow accounts and attach it to user groups.
- C. Create cross-account roles in each account to deny access to the services or actions.
- D. Create a service control policy in the root organizational unit to deny access to the services or actions.

**Architecture Diagram:**

```
 +--------------------------------------------------------+
 |           AWS Organizations — Root OU                 |
 |     Service Control Policy (deny listed actions)      |
 +--------------------------------------------------------+
            |                 |                 |
            v                 v                 v
 +------------------+ +------------------+ +------------------+
 | Account A        | | Account B        | | Account C        |
 | (SCP inherited)  | | (SCP inherited)  | | (SCP inherited)  |
 +------------------+ +------------------+ +------------------+
```

**Correct Answer: D**

**Explanation:** A service control policy (SCP) attached at the root organizational unit applies to every account in the organization and sets a maximum permission boundary, denying the specified services/actions org-wide from one central location. This is scalable (new accounts automatically inherit it) and gives the single point of maintenance the requirement calls for.

- A. Wrong — ACLs (network or S3-style) control access to specific resources at a low level; they aren't a mechanism for centrally restricting AWS service/action access across an entire organization.
- B. Wrong — Security groups are virtual firewalls for EC2/VPC network traffic; they have no concept of "accounts" or IAM service/action permissions.
- C. Wrong — Creating cross-account roles in each account individually means permissions are maintained in many places, not a single point, and doesn't scale well as new accounts are added.

## Question 19

A company needs to retain application log files for a critical application for 10 years. The application team regularly accesses logs from the past month for troubleshooting, but logs older than 1 month are rarely accessed. The application generates more than 10 TB of logs per month.

Which storage option meets these requirements MOST cost-effectively?

- A. Store the logs in Amazon S3. Use AWS Backup to move logs more than 1 month old to S3 Glacier Deep Archive.
- B. Store the logs in Amazon S3. Use S3 Lifecycle policies to move logs more than 1 month old to S3 Glacier Deep Archive.
- C. Store the logs in Amazon CloudWatch Logs. Use AWS Backup to move logs more than 1 month old to S3 Glacier Deep Archive.
- D. Store the logs in Amazon CloudWatch Logs. Use Amazon S3 Lifecycle policies to move logs more than 1 month old to S3 Glacier Deep Archive.

**Architecture Diagram:**

```
 +------------------+   0-1 month    +------------------------+
 | Application logs | -------------> | Amazon S3 Standard      |
 | (10+ TB/month)   |                | (frequent access)       |
 +------------------+                +------------------------+
                                              |
                                              | S3 Lifecycle policy
                                              | after 1 month
                                              v
                                    +------------------------+
                                    | S3 Glacier Deep Archive |
                                    | (rare access, 10 yrs)   |
                                    +------------------------+
```

**Correct Answer: B**

**Explanation:** S3 is the natural store for large log volumes, and S3 Lifecycle policies are the native, built-in mechanism to automatically transition objects to S3 Glacier Deep Archive after a set age — no custom process needed. This directly matches the access pattern (frequent for 1 month, then rare) at the lowest cost for the 10-year retention requirement.

- A. Wrong — AWS Backup is designed for backing up and restoring resources (snapshots of EBS, RDS, DynamoDB, etc.), not for automated storage-class tiering of objects already in S3; S3 Lifecycle policies are the correct native tool for that.
- C. Wrong — CloudWatch Logs is not cost-effective for 10+ TB/month at a 10-year retention (CloudWatch Logs storage is priced much higher than S3), and AWS Backup isn't the mechanism for tiering data to Glacier.
- D. Wrong — Storing 10+ TB/month of logs in CloudWatch Logs long-term is far more expensive than S3; CloudWatch Logs also isn't a direct target for S3 Lifecycle policies without first exporting the logs to S3.

## Question 20

A medical records company is hosting an application on Amazon EC2 instances. The application processes customer data files that are stored on Amazon S3. The EC2 instances are hosted in public subnets. The EC2 instances access Amazon S3 over the internet, but they do not require any other network access.

A new requirement mandates that the network traffic for file transfers take a private route and not be sent over the internet.

Which change to the network architecture should a solutions architect recommend to meet this requirement?

- A. Create a NAT gateway. Configure the route table for the public subnets to send traffic to Amazon S3 through the NAT gateway.
- B. Configure the security group for the EC2 instances to restrict outbound traffic so that only traffic to the S3 prefix list is permitted.
- C. Move the EC2 instances to private subnets. Create a VPC endpoint for Amazon S3, and link the endpoint to the route table for the private subnets.
- D. Remove the internet gateway from the VPC. Set up an AWS Direct Connect connection, and route traffic to Amazon S3 over the Direct Connect connection.

**Architecture Diagram:**

```
 +------------------------+     +--------------------------+     +------------------+
 | EC2 Instances          | --> | VPC Gateway Endpoint     | --> | Amazon S3        |
 | (private subnet)       |     | (for S3, in route table) |     | (private route)  |
 +------------------------+     +--------------------------+     +------------------+
        (no internet gateway / NAT needed for S3 traffic)
```

**Correct Answer: C**

**Explanation:** A VPC gateway endpoint for S3 lets resources in private subnets reach S3 entirely over AWS's private network, without traversing the public internet. Moving the EC2 instances to private subnets and routing S3 traffic through the endpoint fully satisfies "private route, not sent over the internet" — with no NAT gateway, internet gateway, or Direct Connect required.

- A. Wrong — A NAT gateway still routes traffic out through the internet gateway to reach S3's public endpoint; it doesn't make the traffic private, it just hides the instances' direct public IPs.
- B. Wrong — Restricting the security group to the S3 prefix list controls which destinations are allowed, but the traffic itself still travels over the internet since there's no private path configured.
- D. Wrong — Direct Connect provides a private connection from on-premises networks to AWS; it's unnecessary and irrelevant for traffic that both originates and terminates within AWS (EC2 to S3), where a VPC endpoint is the correct, much simpler solution.

## Question 21

A company is concerned about the security of its public web application due to recent web attacks. The application uses an Application Load Balancer (ALB). A solutions architect must reduce the risk of DDoS attacks against the application.

What should the solutions architect do to meet this requirement?

- A. Add an Amazon Inspector agent to the ALB.
- B. Configure Amazon Macie to prevent attacks.
- C. Enable AWS Shield Advanced to prevent attacks.
- D. Configure Amazon GuardDuty to monitor the ALB.

**Architecture Diagram:**

```
 +----------+     +---------------------------------------+     +------------------+
 | Attackers| --> |   AWS Shield Advanced (DDoS protection)| --> | Application Load |
 | / Users  |     |   attached to the ALB                  |     | Balancer (ALB)   |
 +----------+     +---------------------------------------+     +------------------+
```

**Correct Answer: C**

**Explanation:** AWS Shield Advanced is purpose-built to detect and mitigate DDoS attacks against resources like ALBs, CloudFront, and Global Accelerator, including Layer 3/4 and Layer 7 protection along with cost protection and 24/7 DDoS response team access — directly addressing the stated requirement.

- A. Wrong — Amazon Inspector is an automated vulnerability assessment service for EC2/ECR/Lambda; it doesn't attach to an ALB or protect against DDoS attacks.
- B. Wrong — Amazon Macie discovers and protects sensitive data (like PII) in S3; it has no role in preventing DDoS attacks.
- D. Wrong — Amazon GuardDuty is a threat-detection service that analyzes logs for malicious activity and monitors for compromise indicators; it detects and alerts but does not itself mitigate or prevent DDoS attacks the way Shield Advanced does.

## Question 22

A company runs its ecommerce application on AWS. Every new order is published as a message in a RabbitMQ queue that runs on an Amazon EC2 instance in a single Availability Zone. These messages are processed by a different application that runs on a separate EC2 instance. This application stores the details in a PostgreSQL database on another EC2 instance. All the EC2 instances are in the same Availability Zone.

The company needs to redesign its architecture to provide the highest availability with the least operational overhead.

What should a solutions architect do to meet these requirements?

- A. Migrate the queue to a redundant pair (active/standby) of RabbitMQ instances on Amazon MQ. Create a Multi-AZ Auto Scaling group for EC2 instances that host the application. Create another Multi-AZ Auto Scaling group for EC2 instances that host the PostgreSQL database.
- B. Migrate the queue to a redundant pair (active/standby) of RabbitMQ instances on Amazon MQ. Create a Multi-AZ Auto Scaling group for EC2 instances that host the application. Migrate the database to run on a Multi-AZ deployment of Amazon RDS for PostgreSQL.
- C. Create a Multi-AZ Auto Scaling group for EC2 instances that host the RabbitMQ queue. Create another Multi-AZ Auto Scaling group for EC2 instances that host the application. Migrate the database to run on a Multi-AZ deployment of Amazon RDS for PostgreSQL.
- D. Create a Multi-AZ Auto Scaling group for EC2 instances that host the RabbitMQ queue. Create another Multi-AZ Auto Scaling group for EC2 instances that host the application. Create a third Multi-AZ Auto Scaling group for EC2 instances that host the PostgreSQL database.

**Architecture Diagram:**

```
 +--------------------------+     +----------------------------+     +---------------------------+
 | Amazon MQ (RabbitMQ)     | --> | Multi-AZ Auto Scaling      | --> | Amazon RDS for PostgreSQL |
 | active/standby pair      |     | group (application layer)  |     | (Multi-AZ deployment)     |
 +--------------------------+     +----------------------------+     +---------------------------+
```

**Correct Answer: B**

**Explanation:** Replacing self-managed EC2 components with fully managed AWS services minimizes operational overhead while maximizing availability: Amazon MQ provides a managed, Multi-AZ active/standby RabbitMQ broker; a Multi-AZ Auto Scaling group keeps the application layer highly available; and Amazon RDS for PostgreSQL Multi-AZ gives automatic failover for the database, all without the company managing patching, replication, or failover logic itself.

- A. Wrong — Running PostgreSQL on self-managed EC2 instances in an Auto Scaling group doesn't provide a coherent, consistent database (multiple independently-scaled instances don't replicate/synchronize data automatically); RDS Multi-AZ is the correct managed approach.
- C. Wrong — Running RabbitMQ on self-managed EC2 instances in an Auto Scaling group requires manually handling broker clustering/failover, which is significantly more operational overhead than the fully managed Amazon MQ active/standby option.
- D. Wrong — Self-managing all three tiers (queue, application, and database) on EC2 Auto Scaling groups maximizes operational overhead, and Auto Scaling groups don't inherently provide safe, consistent replication for a stateful queue or database.

## Question 23

A company collects data for temperature, humidity, and atmospheric pressure in cities across multiple continents. The average volume of data that the company collects from each site daily is 500 GB. Each site has a high-speed internet connection.

The company wants to aggregate the data from all these global sites as quickly as possible in a single Amazon S3 bucket. The solution must minimize operational complexity.

Which solution meets these requirements?

- A. Turn on S3 Transfer Acceleration on the destination S3 bucket. Use multipart uploads to directly upload site data to the destination S3 bucket.
- B. Upload the data from each site to an S3 bucket in the closest Region. Use S3 Cross-Region Replication to copy objects to the destination S3 bucket. Then remove the data from the origin S3 bucket.
- C. Schedule AWS Snowball Edge Storage Optimized device jobs daily to transfer data from each site to the closest Region. Use S3 Cross-Region Replication to copy objects to the destination S3 bucket.
- D. Upload the data from each site to an Amazon EC2 instance in the closest Region. Store the data in an Amazon Elastic Block Store (Amazon EBS) volume. At regular intervals, take an EBS snapshot and copy it to the Region that contains the destination S3 bucket. Restore the EBS volume in that Region.

**Architecture Diagram:**

```
 +------------------+     +----------------------------+     +------------------------+
 | Global sites     | --> | S3 Transfer Acceleration   | --> | Destination S3 bucket |
 | (high-speed      |     | (edge-optimized upload via |     | (single, aggregated)  |
 |  internet)       |     |  multipart upload)         |     +------------------------+
 +------------------+     +----------------------------+
```

**Correct Answer: A**

**Explanation:** Since every site already has a high-speed internet connection, S3 Transfer Acceleration (which routes uploads through CloudFront edge locations over optimized AWS backbone paths) combined with multipart uploads gets data into the single destination bucket directly and quickly, with essentially just a configuration setting — the least operationally complex option.

- B. Wrong — Uploading to per-Region buckets and then replicating and deleting adds extra steps, extra storage locations, and replication lag/management compared to a direct accelerated upload.
- C. Wrong — Snowball Edge is designed for large offline or bandwidth-constrained transfers; it's unnecessary complexity and slower turnaround when sites already have high-speed internet connections.
- D. Wrong — Manually managing EC2 instances, EBS volumes, snapshots, and cross-Region snapshot copies is a complex, multi-step custom pipeline — the opposite of minimizing operational complexity.

## Question 24 (Choose two)

A company is running a multi-tier web application on premises. The web application is containerized and runs on a number of Linux hosts connected to a PostgreSQL database that contains user records. The operational overhead of maintaining the infrastructure and capacity planning is limiting the company's growth. A solutions architect must improve the application's infrastructure.

Which combination of actions should the solutions architect take to accomplish this? (Choose two.)

- A. Migrate the PostgreSQL database to Amazon Aurora.
- B. Migrate the web application to be hosted on Amazon EC2 instances.
- C. Set up an Amazon CloudFront distribution for the web application content.
- D. Set up Amazon ElastiCache between the web application and the PostgreSQL database.
- E. Migrate the web application to be hosted on AWS Fargate with Amazon Elastic Container Service (Amazon ECS).

**Architecture Diagram:**

```
 +--------------------------------+     +------------------------+
 | AWS Fargate + Amazon ECS       | --> | Amazon Aurora          |
 | (containerized web app,        |     | PostgreSQL-compatible |
 |  no server/capacity mgmt)      |     | (managed, scalable)   |
 +--------------------------------+     +------------------------+
```

**Correct Answers: A and E**

**Explanation:** Migrating the PostgreSQL database to Amazon Aurora (A) removes the operational burden of managing database servers and capacity planning, providing a managed, scalable relational database. Migrating the containerized web application to AWS Fargate with ECS (E) eliminates the need to provision, patch, and capacity-plan the underlying Linux hosts, since Fargate runs containers serverlessly. Together these directly remove the infrastructure and capacity-planning overhead described.

- B. Wrong — Moving to self-managed EC2 instances still requires the company to provision, patch, and capacity-plan servers — it doesn't reduce the operational overhead that's limiting growth.
- C. Wrong — CloudFront improves content delivery performance/latency but does not address the underlying infrastructure management or capacity-planning burden of hosting the app and database.
- D. Wrong — Adding ElastiCache can improve database read performance, but it doesn't remove the operational overhead of maintaining the Linux hosts or database infrastructure itself.

## Question 25

A company is preparing to store confidential data in Amazon S3. For compliance reasons, the data must be encrypted at rest. Encryption key usage must be logged for auditing purposes. Keys must be rotated every year.

Which solution meets these requirements and is the MOST operationally efficient?

- A. Server-side encryption with customer-provided keys (SSE-C)
- B. Server-side encryption with Amazon S3 managed keys (SSE-S3)
- C. Server-side encryption with AWS KMS keys (SSE-KMS) with manual rotation
- D. Server-side encryption with AWS KMS keys (SSE-KMS) with automatic rotation

**Architecture Diagram:**

```
 +------------------+     +--------------------------+     +------------------------+
 | Confidential     | --> | AWS KMS Customer Managed | --> | Amazon S3 (SSE-KMS)   |
 | data uploads     |     | Key (auto-rotates yearly,|     | (encrypted at rest)   |
 |                  |     |  usage logged in         |     +------------------------+
 |                  |     |  CloudTrail)             |
 +------------------+     +--------------------------+
```

**Correct Answer: D**

**Explanation:** SSE-KMS logs every key usage event to AWS CloudTrail automatically, satisfying the auditing requirement, and KMS customer managed keys support automatic annual key rotation with a single setting — no manual process needed. This meets encryption-at-rest, auditability, and yearly rotation with the least operational effort.

- A. Wrong — SSE-C requires the customer to supply and manage the encryption key with every request; there's no AWS-integrated logging of key usage or automatic rotation, making it far more operationally burdensome.
- B. Wrong — SSE-S3 uses AWS-managed keys, but their usage isn't logged per-request in CloudTrail with the same detail as KMS, and there's no user-controlled rotation schedule requirement being met/tracked.
- C. Wrong — Manual rotation requires the company to remember and perform key rotation every year themselves, which is more operationally intensive than KMS's built-in automatic rotation feature.

## Question 26 (Choose two)

A company is migrating its on-premises PostgreSQL database to Amazon Aurora PostgreSQL. The on-premises database must remain online and accessible during the migration. The Aurora database must remain synchronized with the on-premises database.

Which combination of actions must a solutions architect take to meet these requirements? (Choose two.)

- A. Create an ongoing replication task.
- B. Create a database backup of the on-premises database.
- C. Create an AWS Database Migration Service (AWS DMS) replication server.
- D. Convert the database schema by using the AWS Schema Conversion Tool (AWS SCT).
- E. Create an Amazon EventBridge (Amazon CloudWatch Events) rule to monitor the database synchronization.

**Architecture Diagram:**

```
 +------------------------+     +---------------------------+     +------------------------+
 | On-premises PostgreSQL | --> | AWS DMS Replication Server| --> | Amazon Aurora         |
 | (remains online)       |     | + ongoing replication task |     | PostgreSQL            |
 +------------------------+     | (continuous sync)          |     | (stays synchronized)  |
                                +---------------------------+     +------------------------+
```

**Correct Answers: A and C**

**Explanation:** AWS DMS is designed exactly for this scenario — a replication server (C) is provisioned to perform the migration, and an ongoing (continuous) replication task (A) keeps capturing and applying changes from the source to the target after the initial load, so the on-premises database stays online and the Aurora target stays synchronized in near real time.

- B. Wrong — A one-time backup doesn't provide continuous synchronization; it captures a single point in time and wouldn't keep the target updated with ongoing changes.
- D. Wrong — AWS SCT converts database schemas between different engines (e.g., Oracle to PostgreSQL); since PostgreSQL to Aurora PostgreSQL is a homogeneous migration (same engine), schema conversion isn't needed.
- E. Wrong — An EventBridge rule can monitor for events, but it doesn't perform the actual data replication/synchronization work required to keep the databases in sync.

## Question 27

A company runs a stateless web application in production on a group of Amazon EC2 On-Demand Instances behind an Application Load Balancer. The application experiences heavy usage during an 8-hour period each business day. Application usage is moderate and steady overnight. Application usage is low during weekends.

The company wants to minimize its EC2 costs without affecting the availability of the application.

Which solution will meet these requirements?

- A. Use Spot Instances for the entire workload.
- B. Use Reserved Instances for the baseline level of usage. Use Spot instances for any additional capacity that the application needs.
- C. Use On-Demand Instances for the baseline level of usage. Use Spot Instances for any additional capacity that the application needs.
- D. Use Dedicated Instances for the baseline level of usage. Use On-Demand Instances for any additional capacity that the application needs.

**Architecture Diagram:**

```
 +------------------------+     +------------------------------+
 | Reserved Instances     |     | Spot Instances               |
 | (steady baseline:      |     | (extra capacity during the   |
 |  overnight/weekend     |     |  8-hour peak business period)|
 |  usage)                |     +------------------------------+
 +------------------------+
              \___________________________/
                          |
                          v
              +------------------------+
              | Application Load       |
              | Balancer                |
              +------------------------+
```

**Correct Answer: B**

**Explanation:** Reserved Instances offer a significant discount for the steady, predictable baseline usage that runs overnight and on weekends, guaranteeing that capacity is always available. Spot Instances then cover the extra capacity needed only during the predictable 8-hour daily peak, at a steep discount — since the application is stateless, it tolerates Spot interruptions without affecting overall availability, as the Reserved baseline keeps the app running.

- A. Wrong — Relying entirely on Spot Instances risks availability if Spot capacity is reclaimed with no guaranteed baseline capacity to fall back on.
- C. Wrong — On-Demand pricing for the baseline is more expensive than Reserved Instances for guaranteed, steady usage that is known in advance; Reserved Instances are the more cost-effective choice for a predictable baseline.
- D. Wrong — Dedicated Instances address compliance/isolation needs (dedicated hardware) and cost more than Reserved Instances without offering a usage discount; On-Demand for extra capacity is also costlier than Spot for a stateless, interruption-tolerant workload.

## Question 28

A company is running an online transaction processing (OLTP) workload on AWS. This workload uses an unencrypted Amazon RDS DB instance in a Multi-AZ deployment. Daily database snapshots are taken from this instance.

What should a solutions architect do to ensure the database and snapshots are always encrypted moving forward?

- A. Encrypt a copy of the latest DB snapshot. Replace existing DB instance by restoring the encrypted snapshot.
- B. Create a new encrypted Amazon Elastic Block Store (Amazon EBS) volume and copy the snapshots to it. Enable encryption on the DB instance.
- C. Copy the snapshots and enable encryption using AWS Key Management Service (AWS KMS). Restore encrypted snapshot to an existing DB instance.
- D. Copy the snapshots to an Amazon S3 bucket that is encrypted using server-side encryption with AWS Key Management Service (AWS KMS) managed keys (SSE-KMS).

**Architecture Diagram:**

```
 +------------------------+     +---------------------------+     +------------------------+
 | Existing unencrypted   | --> | Encrypted copy of latest  | --> | Restore as new        |
 | RDS DB instance        |     | DB snapshot (KMS)         |     | encrypted DB instance |
 +------------------------+     +---------------------------+     +------------------------+
                                                                       (replaces original,
                                                                        future snapshots
                                                                        inherit encryption)
```

**Correct Answer: A**

**Explanation:** RDS encryption cannot be enabled in place on an existing unencrypted instance. The supported path is to take (or use) a snapshot, create an encrypted copy of that snapshot (via KMS), and then restore a new DB instance from the encrypted snapshot copy. That new instance replaces the original, and from then on all its snapshots are automatically encrypted too.

- B. Wrong — RDS storage isn't a directly user-manageable EBS volume you can swap out this way, and RDS encryption still cannot simply be "enabled" on an existing unencrypted DB instance.
- C. Wrong — You can't restore an encrypted snapshot onto an existing (already-created, unencrypted) DB instance; restoring a snapshot always creates a new DB instance.
- D. Wrong — Copying snapshots into an encrypted S3 bucket encrypts the S3 copies of the data, but it does not encrypt the actual running RDS DB instance or make its ongoing native RDS snapshots encrypted.

## Question 29

A company sells ringtones created from clips of popular songs. The files containing the ringtones are stored in Amazon S3 Standard and are at least 128 KB in size. The company has millions of files, but downloads are infrequent for ringtones older than 90 days. The company needs to save money on storage while keeping the most accessed files readily available for its users.

Which action should the company take to meet these requirements MOST cost-effectively?

- A. Configure S3 Standard-Infrequent Access (S3 Standard-IA) storage for the initial storage tier of the objects.
- B. Move the files to S3 Intelligent-Tiering and configure it to move objects to a less expensive storage tier after 90 days.
- C. Configure S3 inventory to manage objects and move them to S3 Standard-Infrequent Access (S3 Standard-IA) after 90 days.
- D. Implement an S3 Lifecycle policy that moves the objects from S3 Standard to S3 Standard-Infrequent Access (S3 Standard-1A) after 90 days.

**Architecture Diagram:**

```
 +------------------+   0-90 days    +------------------------+
 | Ringtone files   | -------------> | S3 Standard             |
 | (millions,       |                | (frequent access)       |
 |  >=128 KB each)  |                +------------------------+
 +------------------+                          |
                                                | S3 Lifecycle policy
                                                | after 90 days
                                                v
                                      +------------------------+
                                      | S3 Standard-IA          |
                                      | (infrequent access,     |
                                      |  lower storage cost)    |
                                      +------------------------+
```

**Correct Answer: D**

**Explanation:** Since the access pattern is known and predictable (frequent for 90 days, then infrequent), a simple S3 Lifecycle policy transitioning objects to S3 Standard-IA after 90 days achieves the savings with no ongoing monitoring cost. Intelligent-Tiering is better suited to unpredictable access patterns and charges a small monthly monitoring fee per object — with millions of files, that adds unnecessary cost when the pattern is already known.

- A. Wrong — Starting objects directly in S3 Standard-IA would apply infrequent-access pricing (and its per-GB retrieval fee) to newly uploaded files that are actually accessed frequently during their first 90 days, increasing cost.
- B. Wrong — Intelligent-Tiering's per-object monitoring fee is unnecessary overhead when the 90-day access change is already known and predictable — a lifecycle policy achieves the same savings without that fee.
- C. Wrong — S3 Inventory is a reporting tool that lists objects and metadata; it doesn't perform automatic storage-class transitions — S3 Lifecycle policies are the correct mechanism for that.

## Question 30

A company has a highly dynamic batch processing job that uses many Amazon EC2 instances to complete it. The job is stateless in nature, can be started and stopped at any given time with no negative impact, and typically takes upwards of 60 minutes total to complete. The company has asked a solutions architect to design a scalable and cost-effective solution that meets the requirements of the job.

What should the solutions architect recommend?

- A. Implement EC2 Spot Instances.
- B. Purchase EC2 Reserved Instances.
- C. Implement EC2 On-Demand Instances.
- D. Implement the processing on AWS Lambda.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+
 | Batch processing job   | --> | EC2 Spot Instances         |
 | (stateless, tolerant   |     | (deep discount, can be     |
 |  of interruption)      |     |  interrupted/reclaimed)    |
 +------------------------+     +----------------------------+
```

**Correct Answer: A**

**Explanation:** Spot Instances offer the steepest discount off On-Demand pricing and are ideal for workloads that are stateless, can be started/stopped at any time, and tolerate interruption — exactly what this batch job requires. This makes Spot the most cost-effective choice while still being highly scalable.

- B. Wrong — Reserved Instances require a 1- or 3-year commitment for steady-state usage; they don't fit a "highly dynamic" job whose instance needs vary and aren't a continuous baseline.
- C. Wrong — On-Demand Instances work but cost significantly more than Spot for a workload that explicitly tolerates interruption, so they aren't the most cost-effective choice.
- D. Wrong — AWS Lambda has a maximum execution timeout of 15 minutes, which doesn't fit a job that "typically takes upwards of 60 minutes total to complete," making it technically unsuitable.

## Question 31

An application runs on Amazon EC2 instances across multiple Availability Zones. The instances run in an Amazon EC2 Auto Scaling group behind an Application Load Balancer. The application performs best when the CPU utilization of the EC2 instances is at or near 40%.

What should a solutions architect do to maintain the desired performance across all instances in the group?

- A. Use a simple scaling policy to dynamically scale the Auto Scaling group.
- B. Use a target tracking policy to dynamically scale the Auto Scaling group.
- C. Use an AWS Lambda function to update the desired Auto Scaling group capacity.
- D. Use scheduled scaling actions to scale up and scale down the Auto Scaling group.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------------+
 | Amazon CloudWatch      | --> | Target Tracking Policy     | --> | Auto Scaling Group    |
 | (monitors CPU metric)  |     | (target: 40% CPU           |     | (adds/removes          |
 |                        |     |  utilization)               |     |  instances to match)  |
 +------------------------+     +----------------------------+     +------------------------+
```

**Correct Answer: B**

**Explanation:** A target tracking scaling policy lets you specify a target value for a metric (here, 40% CPU utilization), and Auto Scaling automatically adds or removes instances as needed to keep the average at that target — continuously and dynamically, across all instances in the group, with minimal configuration.

- A. Wrong — Simple scaling policies react to a single CloudWatch alarm breach with a fixed adjustment and then wait for a cooldown period, which is less precise and responsive than continuously tracking a target metric value.
- C. Wrong — A custom Lambda function to manually update desired capacity adds unnecessary operational complexity when Auto Scaling's built-in target tracking already handles this natively.
- D. Wrong — Scheduled scaling actions adjust capacity at fixed times, which doesn't respond to the actual, real-time CPU utilization needed to maintain the 40% target.

## Question 32

A global company hosts its web application on Amazon EC2 instances behind an Application Load Balancer (ALB). The web application has static data and dynamic data. The company stores its static data in an Amazon S3 bucket. The company wants to improve performance and reduce latency for the static data and dynamic data. The company is using its own domain name registered with Amazon Route 53.

What should a solutions architect do to meet these requirements?

- A. Create an Amazon CloudFront distribution that has the S3 bucket and the ALB as origins. Configure Route 53 to route traffic to the CloudFront distribution.
- B. Create an Amazon CloudFront distribution that has the ALB as an origin. Create an AWS Global Accelerator standard accelerator that has the S3 bucket as an endpoint. Configure Route 53 to route traffic to the CloudFront distribution.
- C. Create an Amazon CloudFront distribution that has the S3 bucket as an origin. Create an AWS Global Accelerator standard accelerator that has the ALB and the CloudFront distribution as endpoints. Create a custom domain name that points to the accelerator DNS name. Use the custom domain name as an endpoint for the web application.
- D. Create an Amazon CloudFront distribution that has the ALB as an origin. Create an AWS Global Accelerator standard accelerator that has the S3 bucket as an endpoint. Create two domain names. Point one domain name to the CloudFront DNS name for dynamic content. Point the other domain name to the accelerator DNS name for static content. Use the domain names as endpoints for the web application.

**Architecture Diagram:**

```
 +----------+     +----------------------+     +------------------------+
 | Route 53 | --> | Amazon CloudFront    | --> | S3 bucket (origin 1)  |
 | (domain) |     | Distribution         |     | static data            |
 +----------+     | (multi-origin)       |     +------------------------+
                  |                      | --> +------------------------+
                  +----------------------+     | ALB (origin 2)         |
                                                | dynamic data           |
                                                +------------------------+
```

**Correct Answer: A**

**Explanation:** A single CloudFront distribution can have multiple origins — the S3 bucket for static content and the ALB for dynamic content — with path-based behaviors routing each request type to the right origin. CloudFront's edge caching improves performance/latency for both content types, and Route 53 simply routes the company's domain to that one distribution. This is the simplest solution that fully meets the requirement.

- B. Wrong — AWS Global Accelerator does not support Amazon S3 buckets as endpoints (it works with ALBs, NLBs, EC2 instances, and Elastic IPs), so this configuration is not technically valid.
- C. Wrong — Same issue: Global Accelerator can't use an S3 bucket... this option also incorrectly pairs Global Accelerator with a CloudFront distribution as an endpoint, which isn't a standard supported configuration for this use case.
- D. Wrong — Global Accelerator doesn't support S3 as an endpoint, and splitting the application across two separate domain names for static vs. dynamic content adds unnecessary complexity compared to one CloudFront distribution with multiple origins.

## Question 33

A company's containerized application runs on an Amazon EC2 instance. The application needs to download security certificates before it can communicate with other business applications. The company wants a highly secure solution to encrypt and decrypt the certificates in near real time. The solution also needs to store data in highly available storage after the data is encrypted.

Which solution will meet these requirements with the LEAST operational overhead?

- A. Create AWS Secrets Manager secrets for encrypted certificates. Manually update the certificates as needed. Control access to the data by using fine-grained IAM access.
- B. Create an AWS Lambda function that uses the Python cryptography library to receive and perform encryption operations. Store the function in an Amazon S3 bucket.
- C. Create an AWS Key Management Service (AWS KMS) customer managed key. Allow the EC2 role to use the KMS key for encryption operations. Store the encrypted data on Amazon S3.
- D. Create an AWS Key Management Service (AWS KMS) customer managed key. Allow the EC2 role to use the KMS key for encryption operations. Store the encrypted data on Amazon Elastic Block Store (Amazon EBS) volumes.

**Architecture Diagram:**

```
 +------------------------+     +---------------------------+     +------------------------+
 | EC2 instance           | --> | AWS KMS customer managed  | --> | Amazon S3              |
 | (containerized app,    |     | key (encrypt/decrypt via  |     | (encrypted data,       |
 |  IAM role permission)  |     |  EC2 role permission)     |     |  highly available)     |
 +------------------------+     +---------------------------+     +------------------------+
```

**Correct Answer: C**

**Explanation:** AWS KMS provides managed, near-real-time encryption/decryption operations via a simple API call, with the EC2 instance's IAM role granted permission to use the customer managed key — no custom cryptography code to build or maintain. Storing the encrypted data in Amazon S3 satisfies the highly available storage requirement (S3 is designed for 99.999999999% durability across multiple AZs), all with minimal operational overhead.

- A. Wrong — Secrets Manager is designed for storing and rotating secrets like credentials, not for general-purpose near-real-time encryption/decryption of arbitrary certificate data, and "manually update" adds operational overhead.
- B. Wrong — Building a custom Lambda function with a cryptography library means the company must write, secure, and maintain custom encryption code — far more operational overhead than using KMS's managed encryption API.
- D. Wrong — EBS volumes are tied to a single Availability Zone and attach to a single instance at a time, which doesn't meet the "highly available storage" requirement as well as S3 does.

## Question 34

A company recently migrated a message processing system to AWS. The system receives messages into an ActiveMQ queue running on an Amazon EC2 instance. Messages are processed by a consumer application running on Amazon EC2. The consumer application processes the messages and writes results to a MySQL database running on Amazon EC2. The company wants this application to be highly available with low operational complexity.

Which architecture offers the HIGHEST availability?

- A. Add a second ActiveMQ server to another Availability Zone. Add an additional consumer EC2 instance in another Availability Zone. Replicate the MySQL database to another Availability Zone.
- B. Use Amazon MQ with active/standby brokers configured across two Availability Zones. Add an additional consumer EC2 instance in another Availability Zone. Replicate the MySQL database to another Availability Zone.
- C. Use Amazon MQ with active/standby brokers configured across two Availability Zones. Add an additional consumer EC2 instance in another Availability Zone. Use Amazon RDS for MySQL with Multi-AZ enabled.
- D. Use Amazon MQ with active/standby brokers configured across two Availability Zones. Add an Auto Scaling group for the consumer EC2 instances across two Availability Zones. Use Amazon RDS for MySQL with Multi-AZ enabled.

**Architecture Diagram:**

```
 +----------------------------+     +----------------------------+     +---------------------------+
 | Amazon MQ                 | --> | Auto Scaling group        | --> | Amazon RDS for MySQL     |
 | active/standby brokers    |     | (consumer EC2 instances,  |     | Multi-AZ (automatic      |
 | (2 Availability Zones)    |     |  self-healing, 2 AZs)     |     |  failover)                |
 +----------------------------+     +----------------------------+     +---------------------------+
```

**Correct Answer: D**

**Explanation:** Every tier needs both multi-AZ redundancy AND automatic recovery to achieve the highest availability: Amazon MQ active/standby handles automatic broker failover; an Auto Scaling group (not just a single extra instance) automatically replaces failed consumer instances across AZs; and RDS for MySQL Multi-AZ provides automatic database failover. This is also lower operational complexity since Amazon MQ and RDS are managed services.

- A. Wrong — Self-managed ActiveMQ, manually added EC2 instances, and manual MySQL replication all require the company to build and maintain failover logic itself, and none of it auto-recovers from failure — much higher operational complexity and lower availability than managed alternatives.
- B. Wrong — Manually replicating the MySQL database (rather than using RDS Multi-AZ) requires custom replication/failover logic and doesn't provide the same automatic, managed failover as RDS Multi-AZ.
- C. Wrong — Adding just one additional consumer EC2 instance (instead of an Auto Scaling group) means there's no automatic replacement if that instance or its AZ fails, giving lower availability than a self-healing Auto Scaling group.

## Question 35

A company hosts a website analytics application on a single Amazon EC2 On-Demand Instance. The analytics software is written in PHP and uses a MySQL database. The analytics software, the web server that provides PHP, and the database server are all hosted on the EC2 instance. The application is showing signs of performance degradation during busy times and is presenting 5xx errors. The company needs to make the application scale seamlessly.

Which solution will meet these requirements MOST cost-effectively?

- A. Migrate the database to an Amazon RDS for MySQL DB instance. Create an AMI of the web application. Use the AMI to launch a second EC2 On-Demand Instance. Use an Application Load Balancer to distribute the load to each EC2 instance.
- B. Migrate the database to an Amazon RDS for MySQL DB instance. Create an AMI of the web application. Use the AMI to launch a second EC2 On-Demand Instance. Use Amazon Route 53 weighted routing to distribute the load across the two EC2 instances.
- C. Migrate the database to an Amazon Aurora MySQL DB instance. Create an AWS Lambda function to stop the EC2 instance and change the instance type. Create an Amazon CloudWatch alarm to invoke the Lambda function when CPU utilization surpasses 75%.
- D. Migrate the database to an Amazon Aurora MySQL DB instance. Create an AMI of the web application. Apply the AMI to a launch template. Create an Auto Scaling group with the launch template. Configure the launch template to use a Spot Fleet. Attach an Application Load Balancer to the Auto Scaling group.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------------+
 | Application Load       | --> | Auto Scaling group         | --> | Amazon Aurora MySQL   |
 | Balancer                |     | (launch template, Spot     |     | DB instance            |
 |                         |     |  Fleet, scales seamlessly) |     +------------------------+
 +------------------------+     +----------------------------+
```

**Correct Answer: D**

**Explanation:** An Auto Scaling group with a launch template automatically adds or removes instances to match demand, giving true seamless scaling that fixed instance counts (A/B) can't provide. Using a Spot Fleet for those instances significantly lowers compute cost, and migrating to Aurora removes the database bottleneck from the single EC2 instance. An ALB in front of the ASG distributes traffic and integrates natively with scaling.

- A. Wrong — Two fixed EC2 instances behind an ALB is more elastic than one instance, but it's a static count, not "scale seamlessly" with demand, and On-Demand pricing costs more than Spot for this scalable pool.
- B. Wrong — Route 53 weighted routing only splits traffic by fixed percentage weights between two static instances; it doesn't scale capacity up or down and isn't a substitute for an Application Load Balancer with Auto Scaling.
- C. Wrong — Stopping the instance and changing its instance type (vertical scaling) causes downtime during the resize and doesn't provide seamless, continuous scaling the way horizontally scaling an Auto Scaling group does.

## Question 36

A company wants to build a scalable key management infrastructure to support developers who need to encrypt data in their applications.

What should a solutions architect do to reduce the operational burden?

- A. Use multi-factor authentication (MFA) to protect the encryption keys.
- B. Use AWS Key Management Service (AWS KMS) to protect the encryption keys.
- C. Use AWS Certificate Manager (ACM) to create, store, and assign the encryption keys.
- D. Use an IAM policy to limit the scope of users who have access permissions to protect the encryption keys.

**Architecture Diagram:**

```
 +------------------------+     +---------------------------+     +------------------------+
 | Developers /           | --> | AWS Key Management        | --> | Encrypted application  |
 | Applications            |     | Service (managed,        |     | data                    |
 |                        |     |  scalable key store)      |     +------------------------+
 +------------------------+     +---------------------------+
```

**Correct Answer: B**

**Explanation:** AWS KMS is a fully managed service purpose-built for creating, storing, and managing encryption keys at scale, with built-in durability, availability, and integration across AWS services and SDKs — removing the operational burden of building and maintaining custom key management infrastructure.

- A. Wrong — MFA is an authentication mechanism for verifying user identity; it doesn't provide key management infrastructure (creation, storage, rotation) for encryption keys.
- C. Wrong — AWS Certificate Manager is designed for provisioning and managing SSL/TLS certificates, not for general-purpose data encryption key management.
- D. Wrong — An IAM policy controls access permissions but is not itself a key management infrastructure; it doesn't create, store, or manage encryption keys — it would be used alongside KMS, not instead of it.

## Question 37

A company hosts a data lake on AWS. The data lake consists of data in Amazon S3 and Amazon RDS for PostgreSQL. The company needs a reporting solution that provides data visualization and includes all the data sources within the data lake. Only the company's management team should have full access to all the visualizations. The rest of the company should have only limited access.

Which solution will meet these requirements?

- A. Create an analysis in Amazon QuickSight. Connect all the data sources and create new datasets. Publish dashboards to visualize the data. Share the dashboards with the appropriate IAM roles.
- B. Create an analysis in Amazon QuickSight. Connect all the data sources and create new datasets. Publish dashboards to visualize the data. Share the dashboards with the appropriate users and groups.
- C. Create an AWS Glue table and crawler for the data in Amazon S3. Create an AWS Glue extract, transform, and load (ETL) job to produce reports. Publish the reports to Amazon S3. Use S3 bucket policies to limit access to the reports.
- D. Create an AWS Glue table and crawler for the data in Amazon S3. Use Amazon Athena Federated Query to access data within Amazon RDS for PostgreSQL. Generate reports by using Amazon Athena. Publish the reports to Amazon S3. Use S3 bucket policies to limit access to the reports.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | Amazon S3        | --> | Amazon QuickSight    | --> | Dashboards shared with|
 | Amazon RDS       |     | (analysis, datasets, |     | QuickSight users and  |
 | (data sources)   |     |  dashboards)          |     | groups (tiered access)|
 +------------------+     +----------------------+     +------------------------+
```

**Correct Answer: B**

**Explanation:** QuickSight natively connects to both S3 and RDS for PostgreSQL as data sources and is purpose-built for interactive data visualization. Access to published dashboards is managed through QuickSight's own users and groups model — not IAM roles — allowing the management team to be granted full access while everyone else gets limited, tiered access to the same dashboards.

- A. Wrong — QuickSight dashboard sharing is controlled through QuickSight users and groups, not directly through IAM roles; IAM governs AWS account-level permissions, not dashboard-level visualization access.
- C. Wrong — This produces static reports via a Glue ETL job rather than interactive data visualizations, and it doesn't include the RDS PostgreSQL data source at all, missing a full data lake reporting requirement.
- D. Wrong — While Athena Federated Query can incorporate the RDS data, the result is still static reports published to S3 rather than interactive visualizations, and S3 bucket policies provide coarse-grained access control, not the tiered, per-user/group dashboard access QuickSight provides.

## Question 38

A company runs its media rendering application on premises. The company wants to reduce storage costs and has moved all data to Amazon S3. The on-premises rendering application needs low-latency access to storage.

The company needs to design a storage solution for the application. The storage solution must maintain the desired application performance.

Which storage solution will meet these requirements in the MOST cost-effective way?

- A. Use Mountpoint for Amazon S3 to access the data in Amazon S3 for the on-premises application.
- B. Configure an Amazon S3 File Gateway to provide storage for the on-premises application.
- C. Copy the data from Amazon S3 to Amazon FSx for Windows File Server. Configure an Amazon FSx File Gateway to provide storage for the on-premises application.
- D. Configure an on-premises file server. Use the Amazon S3 API to connect to S3 storage. Configure the application to access the storage from the on-premises file server.

**Architecture Diagram:**

```
 +------------------------+     +---------------------------+     +------------------------+
 | On-premises rendering  | --> | S3 File Gateway           | --> | Amazon S3              |
 | application            |     | (local low-latency cache, |     | (durable, low-cost    |
 |                         |     |  backed by S3)             |     |  origin storage)      |
 +------------------------+     +---------------------------+     +------------------------+
```

**Correct Answer: B**

**Explanation:** An S3 File Gateway is a virtual appliance that presents a local file share to the on-premises application while transparently caching frequently accessed data locally for low-latency access, storing the full dataset durably and cost-effectively in S3 behind the scenes — exactly matching "data already in S3, but app needs low-latency access."

- A. Wrong — Mountpoint for Amazon S3 is designed to run on AWS compute (like EC2) to mount S3 as a local file system for cloud-based workloads; it isn't intended for on-premises applications needing local caching for low latency.
- C. Wrong — Copying data from S3 into FSx for Windows File Server duplicates the storage (increasing cost) and adds ongoing synchronization complexity, unlike an S3 File Gateway which keeps S3 as the single source of truth with local caching.
- D. Wrong — Building a custom on-premises file server that calls the S3 API directly means every file access potentially crosses the internet with S3 API latency, and the company would have to build and maintain caching logic itself — far more cost and complexity than the managed S3 File Gateway.

## Question 39

A digital image processing company wants to migrate its on-premises monolithic application to the AWS Cloud. The company processes thousands of images and generates large files as part of the processing workflow.

The company needs a solution to manage the growing number of image processing jobs. The solution must also reduce the manual tasks in the image processing workflow. The company does not want to manage the underlying infrastructure of the solution.

Which solution will meet these requirements with the LEAST operational overhead?

- A. Use Amazon Elastic Container Service (Amazon ECS) with Amazon EC2 Spot Instances to process the images. Configure Amazon Simple Queue Service (Amazon SQS) to orchestrate the workflow. Store the processed files in Amazon Elastic File System (Amazon EFS).
- B. Use AWS Batch jobs to process the images. Use AWS Step Functions to orchestrate the workflow. Store the processed files in an Amazon S3 bucket.
- C. Use AWS Lambda functions and Amazon EC2 Spot Instances to process the images. Store the processed files in Amazon FSx.
- D. Deploy a group of Amazon EC2 instances to process the images. Use AWS Step Functions to orchestrate the workflow. Store the processed files in an Amazon Elastic Block Store (Amazon EBS) volume.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | AWS Step         | --> | AWS Batch jobs        | --> | Amazon S3 bucket       |
 | Functions        |     | (serverless compute,  |     | (processed image      |
 | (orchestrates    |     |  no infra to manage)  |     |  files)                |
 |  workflow)       |     +----------------------+     +------------------------+
 +------------------+
```

**Correct Answer: B**

**Explanation:** AWS Batch dynamically provisions the optimal compute (including Fargate, so no servers to manage) to run image processing jobs at any scale, removing infrastructure management entirely. AWS Step Functions orchestrates the multi-step workflow declaratively, eliminating manual coordination between steps. S3 is the natural, durable, low-management store for the large processed files.

- A. Wrong — Using ECS with EC2 Spot Instances still requires managing the underlying EC2 instances/cluster capacity, which conflicts with "does not want to manage the underlying infrastructure."
- C. Wrong — Managing EC2 Spot Instances directly still requires infrastructure management, and Amazon FSx is a shared file system typically used for specific workloads (e.g., Windows/HPC), not the simplest or most cost-effective store for this use case.
- D. Wrong — Deploying and managing a group of EC2 instances is exactly the infrastructure management the company wants to avoid, and EBS volumes are attached to a single instance, not well-suited as a scalable output store for a distributed job workflow.

## Question 40

A company recently migrated to AWS and wants to implement a solution to protect the traffic that flows in and out of the production VPC. The company had an inspection server in its on-premises data center. The inspection server performed specific operations such as traffic flow inspection and traffic filtering. The company wants to have the same functionalities in the AWS Cloud.

Which solution will meet these requirements?

- A. Use Amazon GuardDuty for traffic inspection and traffic filtering in the production VPC.
- B. Use Traffic Mirroring to mirror traffic from the production VPC for traffic inspection and filtering.
- C. Use AWS Network Firewall to create the required rules for traffic inspection and traffic filtering for the production VPC.
- D. Use AWS Firewall Manager to create the required rules for traffic inspection and traffic filtering for the production VPC.

**Architecture Diagram:**

```
 +------------------+     +----------------------------+     +------------------------+
 | Internet /       | <-> | AWS Network Firewall      | <-> | Production VPC        |
 | on-prem traffic  |     | (stateful inspection &     |     | (resources)            |
 |                  |     |  filtering rules)          |     +------------------------+
 +------------------+     +----------------------------+
```

**Correct Answer: C**

**Explanation:** AWS Network Firewall is a managed, stateful network firewall service designed specifically to inspect and filter traffic flowing in and out of a VPC — directly replacing the on-premises inspection server's traffic flow inspection and filtering functions.

- A. Wrong — Amazon GuardDuty is a threat-detection service that analyzes logs and events for malicious activity; it doesn't perform inline traffic inspection/filtering of network flows.
- B. Wrong — Traffic Mirroring copies network traffic to a monitoring/analysis target for out-of-band inspection (e.g., by third-party tools); it doesn't itself filter or block traffic in-line.
- D. Wrong — AWS Firewall Manager centrally manages firewall rules (such as WAF or Network Firewall policies) across multiple accounts; it's a management layer, not the traffic inspection/filtering engine itself — Network Firewall is the underlying service needed.

## Question 41

A company is building a containerized application on premises and decides to move the application to AWS. The application will have thousands of users soon after it is deployed. The company is unsure how to manage the deployment of containers at scale. The company needs to deploy the containerized application in a highly available architecture that minimizes operational overhead.

Which solution will meet these requirements?

- A. Store container images in an Amazon Elastic Container Registry (Amazon ECR) repository. Use an Amazon Elastic Container Service (Amazon ECS) cluster with the AWS Fargate launch type to run the containers. Use target tracking to scale automatically based on demand.
- B. Store container images in an Amazon Elastic Container Registry (Amazon ECR) repository. Use an Amazon Elastic Container Service (Amazon ECS) cluster with the Amazon EC2 launch type to run the containers. Use target tracking to scale automatically based on demand.
- C. Store container images in a repository that runs on an Amazon EC2 instance. Run the containers on EC2 instances that are spread across multiple Availability Zones. Monitor the average CPU utilization in Amazon CloudWatch. Launch new EC2 instances as needed.
- D. Create an Amazon EC2 Amazon Machine Image (AMI) that contains the container image. Launch EC2 instances in an Auto Scaling group across multiple Availability Zones. Use an Amazon CloudWatch alarm to scale out EC2 instances when the average CPU utilization threshold is breached.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------------+
 | Amazon ECR             | --> | Amazon ECS cluster        | --> | AWS Fargate            |
 | (container images)     |     | (target tracking scaling) |     | (serverless, no        |
 |                         |     |                            |     |  servers to manage)   |
 +------------------------+     +----------------------------+     +------------------------+
```

**Correct Answer: A**

**Explanation:** ECR is the natural managed registry for container images, and ECS with the Fargate launch type runs containers without provisioning or managing any underlying EC2 instances — the lowest operational overhead for "unsure how to manage deployment of containers at scale." Target tracking scaling then automatically adjusts capacity to demand, and ECS spreads tasks across Availability Zones for high availability.

- B. Wrong — The EC2 launch type still requires the company to provision, patch, and scale the underlying EC2 instances/cluster capacity, adding operational overhead that Fargate avoids.
- C. Wrong — Running a self-managed container repository on an EC2 instance and manually launching new instances based on CPU is a largely manual, custom-built solution with significant operational overhead compared to managed ECR/ECS/Fargate.
- D. Wrong — Baking the container image into an AMI and scaling raw EC2 instances abandons the container orchestration benefits (ECS) entirely and requires managing AMI updates and EC2 infrastructure directly — high operational overhead.

## Question 42

A company has a three-tier web application that processes orders from customers. The web tier consists of Amazon EC2 instances behind an Application Load Balancer. The processing tier consists of EC2 instances. The company decoupled the web tier and processing tier by using Amazon Simple Queue Service (Amazon SQS). The storage layer uses Amazon DynamoDB.

At peak times, some users report order processing delays and stalls. The company has noticed that during these delays, the EC2 instances are running at 100% CPU usage, and the SQS queue fills up. The peak times are variable and unpredictable.

The company needs to improve the performance of the application.

Which solution will meet these requirements?

- A. Use scheduled scaling for Amazon EC2 Auto Scaling to scale out the processing tier instances for the duration of peak usage times. Use the CPU Utilization metric to determine when to scale.
- B. Use Amazon ElastiCache for Redis in front of the DynamoDB backend tier. Use target utilization as a metric to determine when to scale.
- C. Add an Amazon CloudFront distribution to cache the responses for the web tier. Use HTTP latency as a metric to determine when to scale.
- D. Use an Amazon EC2 Auto Scaling target tracking policy to scale out the processing tier instances. Use the ApproximateNumberOfMessages attribute to determine when to scale.

**Architecture Diagram:**

```
 +------------------+     +----------------------+     +------------------------+
 | Amazon SQS Queue | --> | Auto Scaling group   | --> | Processing tier EC2   |
 | (backlog metric: |     | (target tracking on  |     | instances (scale out  |
 | ApproximateNumber|     |  queue depth)         |     |  as backlog grows)    |
 | OfMessages)       |     +----------------------+     +------------------------+
 +------------------+
```

**Correct Answer: D**

**Explanation:** The bottleneck is the processing tier being CPU-saturated while the queue backs up, and peak times are variable/unpredictable — so scaling must react dynamically to actual load, not a fixed schedule. A target tracking policy driven by the SQS ApproximateNumberOfMessages attribute scales the processing tier out precisely when the backlog grows, directly addressing the observed bottleneck.

- A. Wrong — Scheduled scaling requires known, fixed times, but the peak times here are explicitly "variable and unpredictable," so a schedule can't reliably anticipate them.
- B. Wrong — Adding a cache in front of DynamoDB doesn't address the described bottleneck, which is 100% CPU usage on the processing tier EC2 instances and a backed-up SQS queue, not a DynamoDB read/write performance issue.
- C. Wrong — Caching web tier responses with CloudFront doesn't help the processing tier, which is where the CPU saturation and queue backlog are actually occurring.

## Question 43

A company wants to migrate its on-premises data center to AWS. According to the company's compliance requirements, the company can use only the ap-northeast-3 Region. Company administrators are not permitted to connect VPCs to the internet.

Which solutions will meet these requirements? (Choose two.)

- A. Use AWS Control Tower to implement data residency guardrails to deny internet access and deny access to all AWS Regions except ap-northeast-3.
- B. Use rules in AWS WAF to prevent internet access. Deny access to all AWS Regions except ap-northeast-3 in the AWS account settings.
- C. Use AWS Organizations to configure service control policies (SCPs) that prevent VPCs from gaining internet access. Deny access to all AWS Regions except ap-northeast-3.
- D. Create an outbound rule for the network ACL in each VPC to deny all traffic from 0.0.0.0/0. Create an IAM policy for each user to prevent the use of any AWS Region other than ap-northeast-3.
- E. Use AWS Config to activate managed rules to detect and alert for internet gateways and to detect and alert for new resources deployed outside of ap-northeast-3.

**Architecture Diagram:**

```
 +--------------------------------------------------------------+
 | AWS Organizations (management account)                        |
 |                                                                |
 |   +----------------------+     +--------------------------+  |
 |   | Control Tower        |     | Service Control Policies |  |
 |   | data residency        |     | - deny VPC internet      |  |
 |   | guardrails             |     |   access                  |  |
 |   | - deny internet access|     | - deny all Regions except|  |
 |   | - deny all Regions    |     |   ap-northeast-3          |  |
 |   |   except ap-northeast-3|    +--------------------------+  |
 |   +----------------------+                                    |
 +--------------------------------------------------------------+
                          |  applies preventively to
                          v
              +--------------------------+
              | Member account VPCs      |
              | (ap-northeast-3 only,    |
              |  no internet gateway)    |
              +--------------------------+
```

**Correct Answer: A, C**

**Explanation:** The requirements demand *preventive*, account-wide guardrails — not just detection — for both internet connectivity and Region usage. AWS Control Tower's data residency guardrails and AWS Organizations SCPs both enforce these restrictions proactively at the organization/account level, blocking VPCs from gaining internet access and blocking use of any Region other than ap-northeast-3 regardless of what individual administrators try to configure.

- B. Wrong — AWS WAF filters HTTP(S) requests to web applications; it has no capability to prevent a VPC from connecting to the internet or to restrict which AWS Regions an account can use.
- D. Wrong — A network ACL outbound deny rule must be configured per VPC (and per subnet) and doesn't stop someone from creating a new VPC with an internet gateway; per-user IAM policies restricting Regions must be applied to every user/role individually and don't prevent an administrator with sufficient permissions from changing or bypassing them, so neither control is a durable, organization-wide guardrail.
- E. Wrong — AWS Config managed rules only detect and alert on noncompliant resources after the fact; they don't prevent internet gateways from being created or prevent deployments outside ap-northeast-3, so this is a detective control, not the required preventive one.

## Question 44

A gaming company is designing a highly available architecture. The application runs on a modified Linux kernel and supports only UDP-based traffic. The company needs the front-end tier to provide the best possible user experience. That tier must have low latency, route traffic to the nearest edge location, and provide static IP addresses for entry into the application endpoints.

What should a solutions architect do to meet these requirements?

- A. Configure Amazon Route 53 to forward requests to an Application Load Balancer. Use AWS Lambda for the application in AWS Application Auto Scaling.
- B. Configure Amazon CloudFront to forward requests to a Network Load Balancer. Use AWS Lambda for the application in an AWS Application Auto Scaling group.
- C. Configure AWS Global Accelerator to forward requests to a Network Load Balancer. Use Amazon EC2 instances for the application in an EC2 Auto Scaling group.
- D. Configure Amazon API Gateway to forward requests to an Application Load Balancer. Use Amazon EC2 instances for the application in an EC2 Auto Scaling group.

**Architecture Diagram:**

```
 +-----------+    UDP    +----------------------+    static IP   +------------------------+
 | Players   | --------> | AWS Global Accelerator| -------------> | Network Load Balancer |
 | (nearest  |  low       | (anycast IPs, routes  |                +------------------------+
 |  edge)    |  latency   |  to nearest edge)     |                          |
 +-----------+           +----------------------+                           v
                                                                  +------------------------+
                                                                  | EC2 Auto Scaling group |
                                                                  | (modified Linux kernel)|
                                                                  +------------------------+
```

**Correct Answer: C**

**Explanation:** AWS Global Accelerator supports both TCP and UDP traffic, provides static anycast IP addresses, and automatically routes each user to the nearest AWS edge location for low latency — matching all three front-end requirements. It pairs with a Network Load Balancer, which operates at the connection level and can forward UDP traffic to backend targets. Because the application requires a modified Linux kernel, it must run on EC2 instances (not Lambda, which doesn't allow custom kernels), so an EC2 Auto Scaling group is the correct compute layer.

- A. Wrong — Route 53 and an Application Load Balancer operate at the HTTP/HTTPS layer and don't support UDP traffic; Lambda also can't run a modified Linux kernel.
- B. Wrong — CloudFront is designed for HTTP/HTTPS content delivery and does not support UDP-based traffic; Lambda again can't host a modified kernel.
- D. Wrong — API Gateway and Application Load Balancers work with HTTP/HTTPS/WebSocket traffic, not UDP, so they cannot carry this application's traffic at all.

## Question 45

A company runs its two-tier ecommerce website on AWS. The web tier consists of a load balancer that sends traffic to Amazon EC2 instances. The database tier uses an Amazon RDS DB instance. The EC2 instances and the RDS DB instance should not be exposed to the public internet. The EC2 instances require internet access to complete payment processing of orders through a third-party web service. The application must be highly available.

Which combination of configuration options will meet these requirements? (Choose two.)

- A. Use an Auto Scaling group to launch the EC2 instances in private subnets. Deploy an RDS Multi-AZ DB instance in private subnets.
- B. Configure a VPC with two private subnets and two NAT gateways across two Availability Zones. Deploy an Application Load Balancer in the private subnets.
- C. Use an Auto Scaling group to launch the EC2 instances in public subnets across two Availability Zones. Deploy an RDS Multi-AZ DB instance in private subnets.
- D. Configure a VPC with one public subnet, one private subnet, and two NAT gateways across two Availability Zones. Deploy an Application Load Balancer in the public subnet.
- E. Configure a VPC with two public subnets, two private subnets, and two NAT gateways across two Availability Zones. Deploy an Application Load Balancer in the public subnets.

**Architecture Diagram:**

```
                              Internet
                                 |
                                 v
        +--------------------------------------------------+
        | Public subnets (AZ-a, AZ-b)                       |
        |   +--------------------+     +------------------+ |
        |   | Application Load   |     | NAT Gateway (x2) | |
        |   | Balancer           |     +------------------+ |
        |   +--------------------+               |          |
        +---------------|--------------------------|--------+
                         v                          v (outbound only)
        +--------------------------------------------------+
        | Private subnets (AZ-a, AZ-b)                      |
        |   +----------------------+   +------------------+ |
        |   | EC2 Auto Scaling grp |   | RDS Multi-AZ     | |
        |   | (payment processing  |   | DB instance      | |
        |   |  via NAT)            |   +------------------+ |
        |   +----------------------+                        |
        +--------------------------------------------------+
```

**Correct Answer: A, E**

**Explanation:** The EC2 instances and RDS instance must stay private while still being highly available and, for EC2, able to reach the internet outbound. Option A keeps EC2 in an Auto Scaling group and RDS in Multi-AZ, both inside private subnets — satisfying "not exposed to the public internet" and high availability for compute and database. Option E completes the picture at the network level: two public subnets (for an internet-facing ALB and the NAT gateways) and two private subnets across two AZs, with two NAT gateways giving the private EC2 instances outbound internet access for payment processing while the ALB in the public subnets receives internet traffic and forwards it inward.

- B. Wrong — An Application Load Balancer that receives traffic from the internet must sit in public subnets; placing it in private subnets prevents it from being reachable as the internet-facing entry point.
- C. Wrong — Launching the EC2 instances in public subnets exposes them directly to the public internet, violating the requirement that they not be publicly exposed.
- D. Wrong — A single public subnet and single private subnet cannot span two Availability Zones, so this configuration fails the high-availability requirement despite provisioning two NAT gateways.

## Question 46

A company hosts its enterprise resource planning (ERP) system in the us-east-1 Region. The system runs on Amazon EC2 instances. Customers use a public API that is hosted on the EC2 instances to exchange information with the ERP system. International customers report slow API response times from their data centers.

Which solution will improve response times for the international customers MOST cost-effectively?

- A. Create an AWS Direct Connect connection that has a public virtual interface (VIF) to provide connectivity from each customer's data center to us-east-1. Route customer API requests by using a Direct Connect gateway to the ERP system API.
- B. Set up an Amazon CloudFront distribution in front of the API. Configure the CachingOptimized managed cache policy to provide improved cache efficiency.
- C. Set up AWS Global Accelerator. Configure listeners for the necessary ports. Configure endpoint groups for the appropriate Regions to distribute traffic. Create an endpoint in the group for the API.
- D. Use AWS Site-to-Site VPN to establish dedicated VPN tunnels between Regions and customer networks. Route traffic to the API over the VPN connections.

**Architecture Diagram:**

```
 +----------------------+      +--------------------------+      +----------------------+
 | International         | ---> | AWS Global Accelerator   | ---> | EC2 instances (API)  |
 | customer data centers |      | (anycast IPs, routes over|      | us-east-1             |
 |                        |      |  AWS global network to    |      +----------------------+
 |                        |      |  nearest healthy endpoint)|
 +----------------------+      +--------------------------+
```

**Correct Answer: C**

**Explanation:** AWS Global Accelerator routes each customer's API traffic onto the AWS global network at the nearest edge location, reducing the number of internet hops and improving response time for international customers reaching a single-Region API — all without provisioning any dedicated per-customer network connections, making it the most cost-effective option.

- A. Wrong — Direct Connect requires a separate dedicated physical connection per customer data center, which is expensive to provision and operate for every international customer, making it far from the most cost-effective choice.
- B. Wrong — The API is used to exchange information with the ERP system, meaning requests and responses are dynamic and not cacheable; a CachingOptimized policy provides little to no benefit for this kind of non-cacheable, transactional traffic.
- D. Wrong — Site-to-Site VPN tunnels are meant for secure private connectivity, not for improving performance, and standard VPN traffic still traverses the public internet end-to-end, so it wouldn't meaningfully reduce latency.

## Question 47

A company runs workloads on AWS. The company needs to connect to a service from an external provider. The service is hosted in the provider's VPC. According to the company's security team, the connectivity must be private and must be restricted to the target service. The connection must be initiated only from the company's VPC.

Which solution will meet these requirements?

- A. Create a VPC peering connection between the company's VPC and the provider's VPC. Update the route table to connect to the target service.
- B. Ask the provider to create a virtual private gateway in its VPC. Use AWS PrivateLink to connect to the target service.
- C. Create a NAT gateway in a public subnet of the company's VPC. Update the route table to connect to the target service.
- D. Ask the provider to create a VPC endpoint for the target service. Use AWS PrivateLink to connect to the target service.

**Architecture Diagram:**

```
 +----------------------+                                   +----------------------+
 | Company's VPC        |                                   | Provider's VPC       |
 |                       |   AWS PrivateLink (private link)  |                       |
 |  +-----------------+ | --------------------------------> | +-----------------+  |
 |  | Interface VPC   | |     connection initiated only     | | VPC endpoint    |  |
 |  | endpoint        | |     from company's VPC            | | service (target |  |
 |  +-----------------+ |                                   | | service only)   |  |
 +----------------------+                                   +----------------------+
```

**Correct Answer: D**

**Explanation:** AWS PrivateLink connects to a specific VPC endpoint service that the provider exposes, so access is scoped only to that target service over private AWS network connectivity, and the interface endpoint in the company's VPC is what initiates the connection — matching all three requirements: private, restricted to the target service, and initiated only from the company's side.

- A. Wrong — VPC peering connects the two entire VPCs and their route tables, giving broad reachability rather than restricting access to just the target service.
- B. Wrong — A virtual private gateway is the VPN/Direct Connect endpoint on the AWS side of a VPC, not a component used to expose a service via PrivateLink; PrivateLink requires the provider to create a VPC endpoint service (backed by a Network Load Balancer), not a virtual private gateway.
- C. Wrong — A NAT gateway provides outbound internet access for private subnets; it doesn't provide private, restricted connectivity to a specific service in another VPC.

## Question 48

A company wants to use high performance computing (HPC) infrastructure on AWS for financial risk modeling. The company's HPC workloads run on Linux. Each HPC workflow runs on hundreds of Amazon EC2 Spot Instances, is short-lived, and generates thousands of output files that are ultimately stored in persistent storage for analytics and long-term future use.

The company seeks a cloud storage solution that permits the copying of on-premises data to long-term persistent storage to make data available for processing by all EC2 instances. The solution should also be a high performance file system that is integrated with persistent storage to read and write datasets and output files.

Which combination of AWS services meets these requirements?

- A. Amazon FSx for Lustre integrated with Amazon S3
- B. Amazon FSx for Windows File Server integrated with Amazon S3
- C. Amazon S3 Glacier integrated with Amazon Elastic Block Store (Amazon EBS)
- D. Amazon S3 bucket with a VPC endpoint integrated with an Amazon Elastic Block Store (Amazon EBS) General Purpose SSD (gp2) volume

**Architecture Diagram:**

```
 +------------------+     copy on-prem data     +----------------------+
 | On-premises data | -------------------------> | Amazon S3            |
 | center            |                            | (long-term          |
 +------------------+                            |  persistent storage)|
                                                   +----------------------+
                                                              |
                                                              | linked file system
                                                              v
                                                   +----------------------+
                                                   | Amazon FSx for       |
                                                   | Lustre (high-perf   |
                                                   | file system)         |
                                                   +----------------------+
                                                              |
                                                              v
                                                   +----------------------+
                                                   | Hundreds of EC2 Spot |
                                                   | Instances (HPC,      |
                                                   | Linux, short-lived)  |
                                                   +----------------------+
```

**Correct Answer: A**

**Explanation:** Amazon FSx for Lustre is a high-performance, Linux-compatible parallel file system purpose-built for HPC workloads, and it natively links to an Amazon S3 bucket, transparently presenting S3 objects as files and writing new output files back to S3 for long-term persistent storage — exactly matching the requirement for a high-performance file system integrated with persistent storage across hundreds of short-lived Spot Instances.

- B. Wrong — Amazon FSx for Windows File Server is built for Windows-based SMB workloads, not Linux HPC file systems, and it does not integrate with S3 the way FSx for Lustre does.
- C. Wrong — S3 Glacier is designed for infrequently accessed archival data with long retrieval times, and EBS volumes are attached to a single EC2 instance at a time, so this pairing doesn't provide a shared, high-performance file system for hundreds of concurrent HPC instances.
- D. Wrong — An EBS gp2 volume can only attach to one EC2 instance (or a small number via Multi-Attach for io1/io2), so it cannot serve as a shared high-performance file system accessible to hundreds of EC2 Spot Instances simultaneously.

## Question 49

A company recently started using Amazon Aurora as the data store for its global ecommerce application. When large reports are run, developers report that the ecommerce application is performing poorly. After reviewing metrics in Amazon CloudWatch, a solutions architect finds that the ReadIOPS and CPUUtilization metrics are spiking when monthly reports run.

What is the MOST cost-effective solution?

- A. Migrate the monthly reporting to Amazon Redshift.
- B. Migrate the monthly reporting to an Aurora Replica.
- C. Migrate the Aurora database to a larger instance class.
- D. Increase the Provisioned IOPS on the Aurora instance.

**Architecture Diagram:**

```
 +----------------------+     writes     +------------------------+
 | Ecommerce application| -------------> | Aurora primary instance |
 +----------------------+                +------------------------+
                                                    |
                                                    | replication
                                                    v
                                          +------------------------+
                                          | Aurora Replica          |
                                          | (handles monthly report |
                                          |  read traffic)          |
                                          +------------------------+
```

**Correct Answer: B**

**Explanation:** The reporting workload is read-heavy and competes with the production application for the same primary instance's CPU and I/O, as shown by ReadIOPS and CPUUtilization spiking together. Routing the monthly reports to an Aurora Replica offloads that read traffic onto a replica that Aurora already keeps in sync at low cost (billed as a normal Aurora instance, no separate data warehouse or migration required), making it the most cost-effective fix.

- A. Wrong — Migrating to Amazon Redshift requires standing up and maintaining an entirely separate data warehouse and an ETL/migration pipeline, which is a much larger cost and operational effort than simply reading from an existing Aurora Replica.
- C. Wrong — Migrating to a larger instance class increases cost continuously to handle a periodic (monthly) spike, paying for capacity that sits idle most of the time.
- D. Wrong — Increasing Provisioned IOPS raises ongoing cost to solve a problem that is really about workload contention on a single instance, not a fundamental lack of available IOPS capacity.

## Question 50

A large media company hosts a web application on AWS. The company wants to start caching confidential media files so that users around the world will have reliable access to the files. The content is stored in Amazon S3 buckets. The company must deliver the content quickly, regardless of where the requests originate geographically.

Which solution will meet these requirements?

- A. Use AWS DataSync to connect the S3 buckets to the web application.
- B. Deploy AWS Global Accelerator to connect the S3 buckets to the web application.
- C. Deploy Amazon CloudFront to connect the S3 buckets to CloudFront edge servers.
- D. Use Amazon Simple Queue Service (Amazon SQS) to connect the S3 buckets to the web application.

**Architecture Diagram:**

```
 +--------------------+     origin     +----------------------+     cached     +------------------+
 | Amazon S3 buckets  | -------------> | Amazon CloudFront    | -------------> | Users worldwide  |
 | (confidential media|                | (edge server cache)  |                | (fast, reliable  |
 |  files)             |                +----------------------+                |  access)          |
 +--------------------+                                                        +------------------+
```

**Correct Answer: C**

**Explanation:** Amazon CloudFront is a content delivery network that caches S3-origin content at edge locations around the world, so requests are served from the nearest edge server rather than traveling back to the origin bucket every time — delivering the confidential media quickly regardless of where users are located, while still supporting signed URLs/cookies to keep the content restricted to authorized users.

- A. Wrong — AWS DataSync is a data transfer service for moving/syncing data between on-premises storage and AWS (or between AWS storage services); it is not a caching or content-delivery mechanism for end users.
- B. Wrong — AWS Global Accelerator improves network performance by routing traffic over the AWS global network to an application endpoint, but it does not cache content at edge locations the way CloudFront does, so it doesn't reduce latency for repeatedly requested static media the way a CDN cache would.
- D. Wrong — Amazon SQS is a message queuing service for decoupling application components; it has no role in caching or delivering media file content to geographically distributed users.

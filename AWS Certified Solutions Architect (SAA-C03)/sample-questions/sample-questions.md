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

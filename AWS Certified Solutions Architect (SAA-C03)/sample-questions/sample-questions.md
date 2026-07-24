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

## Question 51

A company wants to migrate its existing on-premises monolithic application to AWS. The company wants to keep as much of the front-end code and the backend code as possible. However, the company wants to break the application into smaller applications. A different team will manage each application. The company needs a highly scalable solution that minimizes operational overhead.

Which solution will meet these requirements?

- A. Host the application on AWS Lambda. Integrate the application with Amazon API Gateway.
- B. Host the application with AWS Amplify. Connect the application to an Amazon API Gateway API that is integrated with AWS Lambda.
- C. Host the application on Amazon EC2 instances. Set up an Application Load Balancer with EC2 instances in an Auto Scaling group as targets.
- D. Host the application on Amazon Elastic Container Service (Amazon ECS). Set up an Application Load Balancer with Amazon ECS as the target.

**Architecture Diagram:**

```
                         +------------------------+
                         | Application Load        |
                         | Balancer                 |
                         +------------------------+
                              |         |
                 -------------          -------------
                 v                                    v
     +----------------------+              +----------------------+
     | ECS Service (Team A) |              | ECS Service (Team B) |
     | containerized backend|              | containerized backend|
     +----------------------+              +----------------------+
```

**Correct Answer: D**

**Explanation:** Containerizing the existing monolith into services on Amazon ECS preserves most of the existing front-end and backend code (it just needs to be packaged into containers, not rewritten), while still allowing the application to be split into independently deployable services that separate teams can own. ECS with an Application Load Balancer scales each service independently and, especially with the Fargate launch type, minimizes the operational overhead of managing servers.

- A. Wrong — Moving to AWS Lambda requires refactoring the application into individual function handlers that fit Lambda's execution model, which conflicts with the goal of keeping as much of the existing front-end and backend code as possible.
- B. Wrong — This approach also requires rewriting the backend as discrete Lambda functions behind API Gateway, the same invasive refactor problem as option A, and AWS Amplify is oriented around front-end/static hosting rather than preserving an existing backend as-is.
- C. Wrong — Hosting on EC2 instances with an Auto Scaling group can preserve the existing code, but the company still has to patch, manage, and scale the underlying instances itself, which carries more operational overhead than a container platform.

## Question 52

A company is preparing to deploy a new serverless workload. A solutions architect must use the principle of least privilege to configure permissions that will be used to run an AWS Lambda function. An Amazon EventBridge (Amazon CloudWatch Events) rule will invoke the function.

Which solution meets these requirements?

- A. Add an execution role to the function with lambda:InvokeFunction as the action and * as the principal.
- B. Add an execution role to the function with lambda:InvokeFunction as the action and Service: lambda.amazonaws.com as the principal.
- C. Add a resource-based policy to the function with lambda:* as the action and Service: events.amazonaws.com as the principal.
- D. Add a resource-based policy to the function with lambda:InvokeFunction as the action and Service: events.amazonaws.com as the principal.

**Architecture Diagram:**

```
 +----------------------+   lambda:InvokeFunction   +----------------------+
 | Amazon EventBridge   | ------------------------> | AWS Lambda function  |
 | rule                 |   (resource-based policy, |                       |
 | principal: this rule)|    principal: events.     |                       |
 +----------------------+    amazonaws.com)          +----------------------+
```

**Correct Answer: D**

**Explanation:** A Lambda function is invoked by another AWS service through a resource-based policy attached directly to the function, not through the function's own execution role (which governs what the function itself can access). Scoping the policy's action to only lambda:InvokeFunction and the principal to only events.amazonaws.com grants EventBridge exactly the permission it needs and nothing more, satisfying least privilege.

- A. Wrong — An execution role controls what the function can do when it runs, not who is allowed to invoke it; it's also the wrong policy type for granting an external service invoke permission, and using * as the principal would allow anyone to invoke the function.
- B. Wrong — This again attaches the permission to the execution role instead of a resource-based policy, and lambda.amazonaws.com is the Lambda service principal, not EventBridge, so it wouldn't grant EventBridge the ability to invoke the function.
- C. Wrong — Using lambda:* grants every Lambda action (including management actions like UpdateFunctionCode or DeleteFunction) instead of just the invoke permission EventBridge needs, violating least privilege.

## Question 53

A company uses AWS Systems Manager for routine management and patching of Amazon EC2 instances. The EC2 instances are in an IP address type target group behind an Application Load Balancer (ALB).

New security protocols require the company to remove EC2 instances from service during a patch. When the company attempts to follow the security protocol during the next patch, the company receives errors during the patching window.

Which combination of solutions will resolve the errors? (Choose two.)

- A. Change the target type of the target group from IP address type to instance type.
- B. Continue to use the existing Systems Manager document without changes because it is already optimized to handle instances that are in an IP address type target group behind an ALB.
- C. Implement the AWSEC2-PatchLoadBalancerInstance Systems Manager Automation document to manage the patching process.
- D. Use Systems Manager Maintenance Windows to automatically remove the instances from service to patch the instances.
- E. Configure Systems Manager State Manager to remove the instances from service and manage the patching schedule. Use ALB health checks to re-route traffic.

**Architecture Diagram:**

```
 +----------------------+     +------------------------+     +----------------------+
 | Systems Manager      | --> | AWSEC2-Patch          | --> | ALB target group     |
 | Maintenance Window   |     | LoadBalancerInstance   |     | (deregister / patch / |
 | (scheduled trigger)  |     | Automation document    |     |  re-register instance)|
 +----------------------+     +------------------------+     +----------------------+
```

**Correct Answer: C, D**

**Explanation:** The AWSEC2-PatchLoadBalancerInstance Automation document (C) is the purpose-built Systems Manager runbook that determines the load balancer/target group an instance belongs to, verifies its health, deregisters it, waits out connection draining, patches it via AWS-RunPatchBaseline, and re-registers it — replacing whatever generic patching document was producing errors when it tried to remove instances from an ALB-attached target group. Systems Manager Maintenance Windows (D) is the mechanism for scheduling that remove-from-service patching workflow to run automatically across the fleet during a defined maintenance period, which is what the new security protocol requires operationally.

- A. Wrong — The errors are caused by using the wrong patching document/workflow, not by the target group's type; changing the target type from IP address to instance type is an unnecessary and disruptive change to the load balancer configuration when the real fix is switching to the correct Automation document.
- B. Wrong — Continuing with the existing, unmodified Systems Manager document is exactly what is producing the errors during the patching window, so making no change cannot resolve them.
- E. Wrong — State Manager is designed for continuously enforcing a consistent configuration state on instances over time, not for orchestrating a scheduled deregister-patch-re-register workflow against an ALB target group.

## Question 54

A company has implemented a self-managed DNS solution on three Amazon EC2 instances behind a Network Load Balancer (NLB) in the us-west-2 Region. Most of the company's users are located in the United States and Europe. The company wants to improve the performance and availability of the solution. The company launches and configures three EC2 instances in the eu-west-1 Region and adds the EC2 instances as targets for a new NLB.

Which solution can the company use to route traffic to all the EC2 instances?

- A. Create an Amazon Route 53 geolocation routing policy to route requests to one of the two NLBs. Create an Amazon CloudFront distribution. Use the Route 53 record as the distribution's origin.
- B. Create a standard accelerator in AWS Global Accelerator. Create endpoint groups in us-west-2 and eu-west-1. Add the two NLBs as endpoints for the endpoint groups.
- C. Attach Elastic IP addresses to the six EC2 instances. Create an Amazon Route 53 geolocation routing policy to route requests to one of the six EC2 instances. Create an Amazon CloudFront distribution. Use the Route 53 record as the distribution's origin.
- D. Replace the two NLBs with two Application Load Balancers (ALBs). Create an Amazon Route 53 latency routing policy to route requests to one of the two ALBs. Create an Amazon CloudFront distribution. Use the Route 53 record as the distribution's origin.

**Architecture Diagram:**

```
 +----------+     +--------------------------+     +----------------------+
 | Users in | --> | AWS Global Accelerator   | --> | Endpoint group:       |
 | US/Europe|     | (standard accelerator,   |     | us-west-2 -> NLB      |
 +----------+     |  static anycast IPs,     |     +----------------------+
                   |  health-based routing)   |     +----------------------+
                   +--------------------------+ --> | eu-west-1 -> NLB     |
                                                     +----------------------+
```

**Correct Answer: B**

**Explanation:** AWS Global Accelerator is purpose-built to front self-managed, non-HTTP(S) services like a DNS solution (which uses TCP/UDP port 53, not HTTP), providing static anycast IP addresses and routing each user to the nearest healthy Regional endpoint. Configuring a standard accelerator with endpoint groups in us-west-2 and eu-west-1, each pointing at that Region's NLB, lets Global Accelerator route users in the US and Europe to whichever Region gives the best performance while improving availability through automatic failover between Regions.

- A. Wrong — CloudFront is an HTTP(S) content delivery network and cannot front a DNS service running over TCP/UDP port 53, so this solution can't actually carry the application's traffic.
- C. Wrong — Same fundamental problem as A (CloudFront doesn't support non-HTTP DNS traffic), and it also drops the NLBs entirely in favor of directly exposed EC2 instances via Elastic IPs, losing the load-balancing/health-check benefits already in place.
- D. Wrong — Replacing NLBs with ALBs would break the solution, since ALBs operate at Layer 7 (HTTP/HTTPS) and cannot handle the DNS protocol's TCP/UDP traffic that the self-managed DNS service requires; CloudFront in front of it has the same non-HTTP limitation as options A and C.

## Question 55

A company runs a production application on a fleet of Amazon EC2 instances. The application reads the data from an Amazon SQS queue and processes the messages in parallel. The message volume is unpredictable and often has intermittent traffic. This application should continually process messages without any downtime.

Which solution meets these requirements MOST cost-effectively?

- A. Use Spot Instances exclusively to handle the maximum capacity required.
- B. Use Reserved Instances exclusively to handle the maximum capacity required.
- C. Use Reserved Instances for the baseline capacity and use Spot Instances to handle additional capacity.
- D. Use Reserved Instances for the baseline capacity and use On-Demand Instances to handle additional capacity.

**Architecture Diagram:**

```
 +------------------+     +----------------------------------------------+
 | Amazon SQS queue | --> | Auto Scaling group                            |
 | (unpredictable,  |     |  - Reserved Instances: steady baseline load   |
 |  intermittent)   |     |  - Spot Instances: variable extra capacity    |
 +------------------+     +----------------------------------------------+
```

**Correct Answer: C**

**Explanation:** Reserved Instances cover the known, steady baseline workload at a discounted rate, guaranteeing continual message processing without downtime for that portion of capacity. Spot Instances then absorb the unpredictable, intermittent extra traffic at the deepest possible discount; since the fleet processes messages in parallel from a durable SQS queue, any interrupted Spot Instance simply leaves its in-flight messages to become visible again and be picked up by another instance, so occasional Spot interruptions don't cause downtime — making this the most cost-effective combination.

- A. Wrong — Using Spot Instances exclusively to cover maximum capacity risks simultaneous interruptions across the whole fleet during a capacity crunch, which could stall processing entirely and risks downtime.
- B. Wrong — Sizing Reserved Instances to the maximum capacity means paying for peak capacity around the clock even during low-traffic periods, which is far more expensive than necessary for intermittent, unpredictable load.
- D. Wrong — On-Demand Instances cost significantly more than Spot Instances for the variable additional capacity, so this combination is valid for availability but not the most cost-effective option compared to using Spot for the elastic portion.

## Question 56

A company is migrating an application from on-premises servers to Amazon EC2 instances. As part of the migration design requirements, a solutions architect must implement infrastructure metric alarms. The company does not need to take action if CPU utilization increases to more than 50% for a short burst of time. However, if the CPU utilization increases to more than 50% and read IOPS on the disk are high at the same time, the company needs to act as soon as possible. The solutions architect also must reduce false alarms.

What should the solutions architect do to meet these requirements?

- A. Create Amazon CloudWatch composite alarms where possible.
- B. Create Amazon CloudWatch dashboards to visualize the metrics and react to issues quickly.
- C. Create Amazon CloudWatch Synthetics canaries to monitor the application and raise an alarm.
- D. Create single Amazon CloudWatch metric alarms with multiple metric thresholds where possible.

**Architecture Diagram:**

```
 +------------------------+     AND     +------------------------+
 | CloudWatch alarm:      | ----------> | CloudWatch Composite   |
 | CPUUtilization > 50%   |             | Alarm                  |
 +------------------------+             | (triggers only when   |
 +------------------------+     AND     |  both underlying      |
 | CloudWatch alarm:      | ----------> |  alarms are in ALARM) |
 | DiskReadIOPS high      |             +------------------------+
 +------------------------+
```

**Correct Answer: A**

**Explanation:** A CloudWatch composite alarm combines the alarm states of multiple existing metric alarms using AND/OR logic, so it can be configured to go into ALARM only when both the CPUUtilization alarm and the disk read IOPS alarm are simultaneously in ALARM state. This precisely matches the requirement to act only when both conditions occur together, while suppressing the standalone CPU burst that alone shouldn't trigger action — directly reducing false alarms.

- B. Wrong — Dashboards only visualize metrics for humans to look at; they don't evaluate conditions or automatically raise an alarm/take action based on combined metric states.
- C. Wrong — CloudWatch Synthetics canaries run scripted checks against application endpoints (e.g., simulating user traffic); they don't evaluate combined EC2-level metrics like CPU utilization and disk read IOPS together.
- D. Wrong — A single CloudWatch metric alarm can only evaluate one metric at a time; it has no built-in way to combine multiple distinct metric thresholds (like CPU and disk IOPS) into a single AND condition — that capability is exactly what composite alarms add.

## Question 57 (Choose two)

A solutions architect must design a highly available infrastructure for a website. The website is powered by Windows web servers that run on Amazon EC2 instances. The solutions architect must implement a solution that can mitigate a large-scale DDoS attack that originates from thousands of IP addresses. Downtime is not acceptable for the website.

Which actions should the solutions architect take to protect the website from such an attack? (Choose two.)

- A. Use AWS Shield Advanced to stop the DDoS attack.
- B. Configure Amazon GuardDuty to automatically block the attackers.
- C. Configure the website to use Amazon CloudFront for both static and dynamic content.
- D. Use an AWS Lambda function to automatically add attacker IP addresses to VPC network ACLs.
- E. Use EC2 Spot Instances in an Auto Scaling group with a target tracking scaling policy that is set to 80% CPU utilization.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+     +------------------+
 | Attackers| --> | Amazon CloudFront      | --> | AWS Shield Advanced    | --> | EC2 web servers |
 | (1000s of|     | (absorbs/distributes   |     | (detects & mitigates   |     | (Windows,        |
 |  IPs)     |     |  traffic at edge)     |     |  large-scale DDoS)     |     |  no downtime)    |
 +----------+     +------------------------+     +------------------------+     +------------------+
```

**Correct Answers: A, C**

**Explanation:** AWS Shield Advanced provides enhanced, automated DDoS detection and mitigation for large-scale attacks, including Layer 7 protections and integration with AWS WAF, directly addressing the "large-scale DDoS attack from thousands of IP addresses" scenario. Fronting the website with Amazon CloudFront (for both static and dynamic content) absorbs and disperses attack traffic across CloudFront's globally distributed edge network before it ever reaches the origin EC2 instances, and Shield Advanced integrates natively with CloudFront — together keeping the website available with no downtime.

- B. Wrong — Amazon GuardDuty is a detective threat-monitoring service that generates findings and alerts; it does not automatically block traffic or mitigate a DDoS attack on its own.
- D. Wrong — Manually updating VPC network ACLs via a custom Lambda function cannot realistically keep up with an attack originating from thousands of constantly-changing IP addresses, and NACL rule evaluation/management at that scale is slow, error-prone, and not a real-time mitigation mechanism.
- E. Wrong — Scaling EC2 Spot Instances based on CPU utilization does nothing to stop or absorb a DDoS attack; it only reactively adds compute capacity (which Spot Instances can't even guarantee, since they can be interrupted), risking downtime rather than preventing it.

## Question 58

A media company is evaluating the possibility of moving its systems to the AWS Cloud. The company needs at least 10 TB of storage with the maximum possible I/O performance for video processing, 300 TB of very durable storage for storing media content, and 900 TB of storage to meet requirements for archival media that is not in use anymore.

Which set of services should a solutions architect recommend to meet these requirements?

- A. Amazon EBS for maximum performance, Amazon S3 for durable data storage, and Amazon S3 Glacier for archival storage
- B. Amazon EBS for maximum performance, Amazon EFS for durable data storage, and Amazon S3 Glacier for archival storage
- C. Amazon EC2 instance store for maximum performance, Amazon EFS for durable data storage, and Amazon S3 for archival storage
- D. Amazon EC2 instance store for maximum performance, Amazon S3 for durable data storage, and Amazon S3 Glacier for archival storage

**Architecture Diagram:**

```
 +------------------------+   10 TB, max IOPS    +------------------------+
 | Video processing       | -------------------> | Amazon EBS (io2/       |
 | workload                |                       |  Provisioned IOPS)     |
 +------------------------+                       +------------------------+

 +------------------------+   300 TB, durable    +------------------------+
 | Media content storage  | -------------------> | Amazon S3              |
 +------------------------+                       +------------------------+

 +------------------------+   900 TB, archival   +------------------------+
 | Unused archival media  | -------------------> | Amazon S3 Glacier      |
 +------------------------+                       +------------------------+
```

**Correct Answer: A**

**Explanation:** Amazon EBS (Provisioned IOPS volumes) delivers the highest, most consistent I/O performance for a demanding, latency-sensitive workload like video processing on a single instance. Amazon S3 offers extremely high (11 nines) durability at scale for the 300 TB of media content. Amazon S3 Glacier is purpose-built as the low-cost archival tier for the 900 TB of media that is no longer actively used — together this set of services matches each stated capacity and performance requirement to the right storage class.

- B. Wrong — Amazon EFS is a shared, elastic file system optimized for concurrent access by many instances; it does not provide the maximum possible single-workload I/O performance that Provisioned IOPS EBS volumes deliver for video processing.
- C. Wrong — EC2 instance store is ephemeral (data is lost when the instance stops or terminates), so it cannot be relied on for a workload's storage without risking data loss, and EFS/S3 mismatch the required durability and archival characteristics as in option B.
- D. Wrong — EC2 instance store's ephemeral nature makes it unsuitable as the "maximum performance" storage choice when durability of the processed workload matters; a persistent, high-IOPS EBS volume is the appropriate service instead.

## Question 59

A company has two applications: a sender application that sends messages with payloads to be processed and a processing application intended to receive the messages with payloads. The company wants to implement an AWS service to handle messages between the two applications. The sender application can send about 1,000 messages each hour. The messages may take up to 2 days to be processed. If the messages fail to process, they must be retained so that they do not impact the processing of any remaining messages.

Which solution meets these requirements and is the MOST operationally efficient?

- A. Set up an Amazon EC2 instance running a Redis database. Configure both applications to use the instance. Store, process, and delete the messages, respectively.
- B. Use an Amazon Kinesis data stream to receive the messages from the sender application. Integrate the processing application with the Kinesis Client Library (KCL).
- C. Integrate the sender and processor applications with an Amazon Simple Queue Service (Amazon SQS) queue. Configure a dead-letter queue to collect the messages that failed to process.
- D. Subscribe the processing application to an Amazon Simple Notification Service (Amazon SNS) topic to receive notifications to process. Integrate the sender application to write to the SNS topic.

**Architecture Diagram:**

```
 +--------------------+     +------------------------+     +------------------------+
 | Sender application | --> | Amazon SQS queue       | --> | Processing application |
 +--------------------+     +------------------------+     +------------------------+
                                       |
                                       | repeated failures
                                       v
                            +------------------------+
                            | Dead-letter queue       |
                            | (retains failed         |
                            |  messages, isolated)    |
                            +------------------------+
```

**Correct Answer: C**

**Explanation:** Amazon SQS is a fully managed queue that decouples the sender and processing applications with no infrastructure to run, and its message retention (up to 14 days) comfortably covers messages that take up to 2 days to process. Configuring a dead-letter queue automatically isolates messages that repeatedly fail processing, so they don't block or interfere with the processing of remaining messages — meeting every requirement with a native, managed feature and the least operational effort.

- A. Wrong — Running Redis on a self-managed EC2 instance requires provisioning, patching, scaling, and building custom logic for storing/processing/deleting messages and handling failures, which is far more operational overhead than a managed SQS queue with a DLQ.
- B. Wrong — Kinesis Data Streams is designed for high-throughput, ordered streaming data with consumers tracking their own position; it doesn't have a native concept of per-message failure retention like an SQS dead-letter queue, and it adds unnecessary operational complexity (shard management, KCL) for a modest 1,000-messages-per-hour workload.
- D. Wrong — SNS is a push-based pub/sub notification service without built-in message retention/redelivery for slow (up to 2-day) processing or a dead-letter mechanism for isolating failed messages; it isn't designed for durable, queued point-to-point message processing.

## Question 60

A company runs an environment where data is stored in an Amazon S3 bucket. The objects are accessed frequently throughout the day. The company has strict data encryption requirements for data that is stored in the S3 bucket. The company currently uses AWS Key Management Service (AWS KMS) for encryption.

The company wants to optimize costs associated with encrypting S3 objects without making additional calls to AWS KMS.

Which solution will meet these requirements?

- A. Use server-side encryption with Amazon S3 managed keys (SSE-S3).
- B. Use an S3 Bucket Key for server-side encryption with AWS KMS keys (SSE-KMS) on the new objects.
- C. Use client-side encryption with AWS KMS customer managed keys.
- D. Use server-side encryption with customer-provided keys (SSE-C) stored in AWS KMS.

**Architecture Diagram:**

```
 +------------------+     +------------------------+     +------------------------+
 | S3 requests      | --> | S3 Bucket Key           | --> | AWS KMS               |
 | (frequent access)|     | (cached data key,       |     | (called only to       |
 |                   |     |  reused for many        |     |  refresh the bucket   |
 |                   |     |  object operations)     |     |  key periodically)    |
 +------------------+     +------------------------+     +------------------------+
```

**Correct Answer: B**

**Explanation:** An S3 Bucket Key generates a short-lived bucket-level key from the KMS key and caches it inside S3, so most subsequent SSE-KMS encrypt/decrypt operations are handled using that cached key instead of making a fresh call to AWS KMS for every object request. This dramatically reduces KMS API call volume (and the associated per-request cost) for frequently accessed objects while still meeting the strict encryption requirement, since the underlying encryption still relies on AWS KMS.

- A. Wrong — SSE-S3 uses Amazon-managed keys instead of AWS KMS entirely, so while it avoids KMS calls, it drops the company's existing KMS-based key management and doesn't meet the requirement to keep using AWS KMS for encryption.
- C. Wrong — Client-side encryption with KMS customer managed keys still requires a KMS API call (GenerateDataKey) for every object encrypted or decrypted on the client, so it doesn't reduce KMS calls or cost — if anything it adds application complexity.
- D. Wrong — SSE-C uses keys that the customer supplies with each request and that AWS does not store or manage at all; AWS KMS cannot store SSE-C keys, so this option describes a combination that isn't how SSE-C or KMS actually work.

## Question 61

A company's web application is running on Amazon EC2 instances behind an Application Load Balancer. The company recently changed its policy, which now requires the application to be accessed from one specific country only.

Which configuration will meet this requirement?

- A. Configure the security group for the EC2 instances.
- B. Configure the security group on the Application Load Balancer.
- C. Configure AWS WAF on the Application Load Balancer in a VPC.
- D. Configure the network ACL for the subnet that contains the EC2 instances.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+
 | Requests | --> | AWS WAF (geo-match     | --> | Application Load       |
 | (global) |     |  rule: allow only one  |     | Balancer               |
 +----------+     |  specific country)     |     +------------------------+
                   +------------------------+
```

**Correct Answer: C**

**Explanation:** AWS WAF supports geographic match (geo-match) conditions that inspect the source country of each request and allow or block traffic based on it. Attaching a WAF web ACL with a geo-match rule to the Application Load Balancer restricts application access to only the one approved country, which is exactly the kind of Layer 7, country-based filtering that security groups and network ACLs cannot perform.

- A. Wrong — Security groups filter traffic only by IP address/CIDR range and port; they have no concept of geographic/country-based origin, so they can't enforce a single-country restriction.
- B. Wrong — Same limitation as option A: security groups (even on the ALB) can only match IP ranges and ports, not the request's country of origin.
- D. Wrong — Network ACLs also filter based on IP address ranges and ports at the subnet level; they have no built-in geographic awareness and can't reliably restrict access to a single country.

## Question 62

A company needs to store data in Amazon S3 and must prevent the data from being changed. The company wants new objects that are uploaded to Amazon S3 to remain unchangeable for a nonspecific amount of time until the company decides to modify the objects. Only specific users in the company's AWS account can have the ability to delete the objects.

What should a solutions architect do to meet these requirements?

- A. Create an S3 Glacier vault. Apply a write-once, read-many (WORM) vault lock policy to the objects.
- B. Create an S3 bucket with S3 Object Lock enabled. Enable versioning. Set a retention period of 100 years. Use governance mode as the S3 bucket's default retention mode for new objects.
- C. Create an S3 bucket. Use AWS CloudTrail to track any S3 API events that modify the objects. Upon notification, restore the modified objects from any backup versions that the company has.
- D. Create an S3 bucket with S3 Object Lock enabled. Enable versioning. Add a legal hold to the objects. Add the s3:PutObjectLegalHold permission to the IAM policies of users who need to delete the objects.

**Architecture Diagram:**

```
 +------------------+     +------------------------+     +------------------------+
 | New object       | --> | S3 bucket              | --> | S3 Object Lock:        |
 | upload            |     | (versioning enabled)   |     | Legal Hold (ON)        |
 +------------------+     +------------------------+     | - blocks all changes/  |
                                                            |   deletes indefinitely|
                                                            | - only users with     |
                                                            |   s3:PutObjectLegalHold|
                                                            |   can remove the hold |
                                                            +------------------------+
```

**Correct Answer: D**

**Explanation:** An S3 Object Lock legal hold protects an object for an indefinite, nonspecific duration — it has no expiration date and stays in place until someone with permission explicitly removes it, unlike a fixed retention period. Granting the s3:PutObjectLegalHold permission only to specific IAM users lets exclusively those users remove the legal hold (and thus subsequently delete the object), while everyone else is blocked from changing or deleting it, matching both requirements exactly.

- A. Wrong — S3 Glacier vault lock with a WORM policy applies to a Glacier vault, not standard S3 objects, and is intended for fixed compliance archiving, not general S3 storage with the flexibility to later allow specific users to delete objects.
- B. Wrong — A 100-year retention period is a specific, fixed duration, not the "nonspecific amount of time" the company wants, and governance mode still allows users with special permissions to bypass or shorten the retention entirely — it doesn't tie deletion ability to specific users as precisely as a legal hold combined with scoped IAM permissions.
- C. Wrong — This is a reactive, detect-and-restore approach that doesn't actually prevent objects from being changed in the first place; it only notices changes after they've already happened and relies on backups existing, which doesn't meet the "remain unchangeable" requirement.

## Question 63 (Choose two)

A company is running a publicly accessible serverless application that uses Amazon API Gateway and AWS Lambda. The application's traffic recently spiked due to fraudulent requests from botnets.

Which steps should a solutions architect take to block requests from unauthorized users? (Choose two.)

- A. Create a usage plan with an API key that is shared with genuine users only.
- B. Integrate logic within the Lambda function to ignore the requests from fraudulent IP addresses.
- C. Implement an AWS WAF rule to target malicious requests and trigger actions to filter them out.
- D. Convert the existing public API to a private API. Update the DNS records to redirect users to the new API endpoint.
- E. Create an IAM role for each user attempting to access the API. A user will assume the role when making the API call.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+     +------------------+
 | Botnets  | -x  | AWS WAF                | --> | API Gateway            | --> | AWS Lambda       |
 | Genuine  | --> | (blocks malicious      |     | (usage plan + API key  |     +------------------+
 | users    |     |  request patterns)     |     |  restricts to genuine  |
 +----------+     +------------------------+     |  users only)           |
                                                    +------------------------+
```

**Correct Answers: A, C**

**Explanation:** A usage plan with an API key (A) requires every request to present a valid key, so only genuine users who were issued a key can call the API, immediately blocking anonymous botnet traffic. AWS WAF (C), attached to API Gateway, can inspect incoming requests and apply rules (rate-based rules, IP reputation lists, bot control) to detect and filter out malicious traffic patterns before it reaches the API. Together these block unauthorized/fraudulent requests at the edge with native, managed features.

- B. Wrong — Hardcoding fraudulent IP-blocking logic inside the Lambda function still invokes Lambda for every fraudulent request (incurring cost and load) and is a fragile, manually maintained blocklist compared to AWS WAF's purpose-built filtering.
- D. Wrong — Converting to a private API removes public accessibility entirely, which conflicts with the requirement that the application remains a "publicly accessible" application; it doesn't selectively block only unauthorized users.
- E. Wrong — Requiring an IAM role per user is impractical for a publicly accessible application with external/anonymous users and isn't how public-facing API authorization is typically implemented; it doesn't address filtering out botnet traffic.

## Question 64

A company uses a three-tier web application to provide training to new employees. The application is accessed for only 12 hours every day. The company is using an Amazon RDS for MySQL DB instance to store information and wants to minimize costs.

What should a solutions architect do to meet these requirements?

- A. Configure an IAM policy for AWS Systems Manager Session Manager. Create an IAM role for the policy. Update the trust relationship of the role. Set up automatic start and stop for the DB instance.
- B. Create an Amazon ElastiCache for Redis cache cluster that gives users the ability to access the data from the cache when the DB instance is stopped. Invalidate the cache after the DB instance is started.
- C. Launch an Amazon EC2 instance. Create an IAM role that grants access to Amazon RDS. Attach the role to the EC2 instance. Configure a cron job to start and stop the EC2 instance on the desired schedule.
- D. Create AWS Lambda functions to start and stop the DB instance. Create Amazon EventBridge (Amazon CloudWatch Events) scheduled rules to invoke the Lambda functions. Configure the Lambda functions as event targets for the rules.

**Architecture Diagram:**

```
 +------------------------+     invoke     +------------------------+     start/stop     +------------------+
 | Amazon EventBridge     | -------------> | AWS Lambda functions   | -----------------> | Amazon RDS       |
 | scheduled rules         |                | (start DB, stop DB)    |                     | for MySQL        |
 | (e.g. 12h on/12h off)  |                +------------------------+                     +------------------+
 +------------------------+
```

**Correct Answer: D**

**Explanation:** Scheduling AWS Lambda functions via Amazon EventBridge rules to start the DB instance at the beginning of the 12-hour usage window and stop it at the end directly stops the company from paying for RDS compute while the application is idle, using only fully managed, serverless components with no infrastructure to run — the lowest-cost, lowest-effort way to meet the requirement.

- A. Wrong — AWS Systems Manager Session Manager provides secure shell/console access to managed instances; it has no feature for scheduling automatic start/stop of an RDS DB instance, so this option doesn't actually implement the required behavior.
- B. Wrong — Adding an ElastiCache cluster increases cost rather than reducing it, and it doesn't stop the RDS instance from running (or being billed) during idle hours — it addresses a different problem (read performance) entirely.
- C. Wrong — This starts and stops an EC2 instance, not the RDS DB instance itself, so the RDS instance would keep running (and incurring cost) around the clock regardless of the EC2 schedule.

## Question 65

A company runs an application in a private subnet behind an Application Load Balancer (ALB) in a VPC. The VPC has a NAT gateway and an internet gateway. The application calls the Amazon S3 API to store objects.

According to the company's security policy, traffic from the application must not travel across the internet.

Which solution will meet these requirements MOST cost-effectively?

- A. Configure an S3 interface endpoint. Create a security group that allows outbound traffic to Amazon S3.
- B. Configure an S3 gateway endpoint. Update the VPC route table to use the endpoint.
- C. Configure an S3 bucket policy to allow traffic from the Elastic IP address that is assigned to the NAT gateway.
- D. Create a second NAT gateway in the same subnet where the legacy application is deployed. Update the VPC route table to use the second NAT gateway.

**Architecture Diagram:**

```
 +------------------------+     +--------------------------+     +------------------+
 | Application            | --> | S3 Gateway Endpoint      | --> | Amazon S3        |
 | (private subnet)       |     | (VPC route table entry,  |     | (private route,  |
 +------------------------+     |  no hourly/data charge)  |     |  no internet)    |
                                +--------------------------+     +------------------+
```

**Correct Answer: B**

**Explanation:** An S3 gateway endpoint routes traffic to Amazon S3 entirely over AWS's private network by adding a route table entry, keeping traffic off the internet, and unlike an interface endpoint it has no hourly or per-GB processing charge — making it both compliant with the no-internet policy and the most cost-effective of the private-connectivity options.

- A. Wrong — An S3 interface endpoint (a PrivateLink ENI) also keeps traffic private, but it incurs hourly and data processing charges, making it more expensive than a gateway endpoint for the same requirement.
- C. Wrong — Restricting the S3 bucket policy to the NAT gateway's Elastic IP still routes traffic out through the NAT gateway and internet gateway to reach S3's public endpoint, meaning the traffic still traverses the internet path, violating the policy.
- D. Wrong — Adding a second NAT gateway still sends S3 traffic out through a NAT/internet gateway to the public S3 endpoint, which doesn't satisfy the "must not travel across the internet" requirement, and it adds unnecessary extra cost.

## Question 66 (Choose two)

A company uses a popular content management system (CMS) for its corporate website. However, the required patching and maintenance are burdensome. The company is redesigning its website and wants a new solution. The website will be updated four times a year and does not need to have any dynamic content available. The solution must provide high scalability and enhanced security.

Which combination of changes will meet these requirements with the LEAST operational overhead? (Choose two.)

- A. Configure Amazon CloudFront in front of the website to use HTTPS functionality.
- B. Deploy an AWS WAF web ACL in front of the website to provide HTTPS functionality.
- C. Create and deploy an AWS Lambda function to manage and serve the website content.
- D. Create the new website and an Amazon S3 bucket. Deploy the website on the S3 bucket with static website hosting enabled.
- E. Create the new website. Deploy the website by using an Auto Scaling group of Amazon EC2 instances behind an Application Load Balancer.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+
 | Visitors | --> | Amazon CloudFront      | --> | Amazon S3 bucket       |
 |          |     | (HTTPS, edge caching,  |     | (static website        |
 |          |     |  scalability, security)|     |  hosting, no servers)  |
 +----------+     +------------------------+     +------------------------+
```

**Correct Answers: A, D**

**Explanation:** Since the website has no dynamic content and updates only four times a year, hosting it as a static website on Amazon S3 (D) eliminates all server patching/maintenance — S3 durably serves the HTML/CSS/JS files directly with virtually unlimited scalability and no infrastructure to manage. Placing Amazon CloudFront in front of it (A) adds HTTPS termination, edge caching for further scalability/performance, and an additional security layer (e.g., restricting direct S3 access via OAI) — together fully satisfying scalability, security, and minimal operational overhead.

- B. Wrong — AWS WAF filters malicious request patterns (SQLi, XSS, rate limiting); it is not a mechanism for providing HTTPS/TLS termination, so it cannot fulfill the HTTPS functionality on its own.
- C. Wrong — Using Lambda to manage and serve static website content adds unnecessary custom code and operational complexity compared to simply hosting static files on S3, which requires no code at all for this use case.
- E. Wrong — An Auto Scaling group of EC2 instances behind an ALB still requires patching the underlying OS/CMS software and managing server infrastructure, which is exactly the operational burden the company is trying to eliminate.

## Question 67

A company wants to run applications in containers in the AWS Cloud. These applications are stateless and can tolerate disruptions within the underlying infrastructure. The company needs a solution that minimizes cost and operational overhead.

What should a solutions architect do to meet these requirements?

- A. Use Spot Instances in an Amazon EC2 Auto Scaling group to run the application containers.
- B. Use Spot Instances in an Amazon Elastic Kubernetes Service (Amazon EKS) managed node group.
- C. Use On-Demand Instances in an Amazon EC2 Auto Scaling group to run the application containers.
- D. Use On-Demand Instances in an Amazon Elastic Kubernetes Service (Amazon EKS) managed node group.

**Architecture Diagram:**

```
 +------------------------+     +------------------------+
 | Amazon EC2 Auto        | --> | Spot Instances          |
 | Scaling group           |     | (stateless containers, |
 | (runs containers)       |     |  tolerate disruption,  |
 +------------------------+     |  lowest cost)           |
                                  +------------------------+
```

**Correct Answer: A**

**Explanation:** Since the applications are stateless and can tolerate infrastructure disruptions, Spot Instances are the cheapest compute option and fit this workload well without risking availability problems. Running containers directly in an EC2 Auto Scaling group avoids the extra control-plane cost and operational complexity of managing a Kubernetes cluster, so this combination minimizes both cost and operational overhead compared to the EKS-based options.

- B. Wrong — Amazon EKS adds a per-cluster control plane cost and the operational overhead of managing Kubernetes (upgrades, add-ons, cluster configuration), which is more than needed when a simpler EC2 Auto Scaling group can run the same containers just as effectively.
- C. Wrong — On-Demand Instances cost more than Spot Instances; since the workload is stateless and disruption-tolerant, there's no need to pay the On-Demand premium.
- D. Wrong — This combines the higher cost of On-Demand Instances with the added operational overhead of managing an EKS cluster, making it the least cost- and overhead-optimized of the four options.

## Question 68

A company is implementing a new business application. The application runs on two Amazon EC2 instances and uses an Amazon S3 bucket for document storage. A solutions architect needs to ensure that the EC2 instances can access the S3 bucket.

What should the solutions architect do to meet this requirement?

- A. Create an IAM role that grants access to the S3 bucket. Attach the role to the EC2 instances.
- B. Create an IAM policy that grants access to the S3 bucket. Attach the policy to the EC2 instances.
- C. Create an IAM group that grants access to the S3 bucket. Attach the group to the EC2 instances.
- D. Create an IAM user that grants access to the S3 bucket. Attach the user account to the EC2 instances.

**Architecture Diagram:**

```
 +------------------------+     +------------------------+     +------------------+
 | EC2 instances          | --> | IAM role (instance     | --> | Amazon S3        |
 | (instance profile)     |     |  profile, S3 access    |     | bucket           |
 +------------------------+     |  policy attached)       |     +------------------+
                                +------------------------+
```

**Correct Answer: A**

**Explanation:** An IAM role is the only one of these IAM constructs that can be attached directly to an EC2 instance (via an instance profile), automatically supplying temporary credentials to the instance so it can access the S3 bucket without any hardcoded long-term credentials.

- B. Wrong — An IAM policy defines permissions but cannot be attached directly to an EC2 instance by itself; it must be attached to an identity such as a role, user, or group.
- C. Wrong — IAM groups can only contain IAM users, not EC2 instances, so a group cannot be attached to an instance to grant it access.
- D. Wrong — IAM users represent people or applications with long-term credentials and cannot be "attached" to an EC2 instance; embedding user credentials on an instance is also an insecure practice compared to using a role.

## Question 69

A company's production environment consists of Amazon EC2 On-Demand Instances that run constantly between Monday and Saturday. The instances must run for only 12 hours on Sunday and cannot tolerate interruptions. The company wants to cost-optimize the production environment.

Which solution will meet these requirements MOST cost-effectively?

- A. Purchase Scheduled Reserved Instances for the EC2 instances that run for only 12 hours on Sunday. Purchase Standard Reserved Instances for the EC2 instances that run constantly between Monday and Saturday.
- B. Purchase Convertible Reserved Instances for the EC2 instances that run for only 12 hours on Sunday. Purchase Standard Reserved Instances for the EC2 instances that run constantly between Monday and Saturday.
- C. Use Spot Instances for the EC2 instances that run for only 12 hours on Sunday. Purchase Standard Reserved Instances for the EC2 instances that run constantly between Monday and Saturday.
- D. Use Spot Instances for the EC2 instances that run for only 12 hours on Sunday. Purchase Convertible Reserved Instances for the EC2 instances that run constantly between Monday and Saturday.

**Architecture Diagram:**

```
 +------------------------------+     +------------------------------+
 | Mon-Sat: constant workload   | --> | Standard Reserved Instances  |
 | (steady, predictable)        |     | (deepest discount, full-time)|
 +------------------------------+     +------------------------------+

 +------------------------------+     +------------------------------+
 | Sunday: 12-hour recurring    | --> | Scheduled Reserved Instances |
 | workload (fixed schedule,    |     | (matches recurring schedule, |
 |  no interruptions allowed)   |     |  guaranteed capacity)        |
 +------------------------------+     +------------------------------+
```

**Correct Answer: A**

**Explanation:** The Monday-through-Saturday workload runs constantly and predictably, so Standard Reserved Instances give the deepest discount for that always-on capacity. The Sunday workload has a fixed, recurring 12-hour schedule and cannot tolerate interruptions, which is exactly the use case Scheduled Reserved Instances are designed for — they reserve capacity for a specific recurring time window at a discount versus On-Demand, while still guaranteeing the instance is available and uninterrupted during that window.

- B. Wrong — Convertible Reserved Instances allow changing instance attributes over the term but, like Standard RIs, are billed for a full-time commitment; they aren't priced or structured around a recurring partial-day schedule the way Scheduled Reserved Instances are, making them less cost-effective for the 12-hour Sunday workload.
- C. Wrong — Spot Instances can be interrupted by AWS at any time, which directly violates the requirement that the Sunday workload "cannot tolerate interruptions."
- D. Wrong — Same Spot interruption problem as option C for the Sunday workload, and Convertible Reserved Instances are unnecessary (and typically no cheaper than Standard RIs) for the constant, unchanging Monday-Saturday workload described.

## Question 70

A company needs to save the results from a medical trial to an Amazon S3 repository. The repository must allow a few scientists to add new files and must restrict all other users to read-only access. No users can have the ability to modify or delete any files in the repository. The company must keep every file in the repository for a minimum of 1 year after its creation date.

Which solution will meet these requirements?

- A. Use S3 Object Lock in governance mode with a legal hold of 1 year.
- B. Use S3 Object Lock in compliance mode with a retention period of 365 days.
- C. Use an IAM role to restrict all users from deleting or changing objects in the S3 bucket. Use an S3 bucket policy to only allow the IAM role.
- D. Configure the S3 bucket to invoke an AWS Lambda function every time an object is added. Configure the function to track the hash of the saved object so that modified objects can be marked accordingly.

**Architecture Diagram:**

```
 +------------------+     +------------------------+     +------------------------+
 | Scientists       | --> | Amazon S3 bucket        | --> | S3 Object Lock          |
 | (add new files)  |     | (versioning enabled)    |     | Compliance mode          |
 +------------------+     +------------------------+     | retention: 365 days     |
                                                            | (no user, incl. root,   |
                                                            |  can delete/modify)     |
                                                            +------------------------+
```

**Correct Answer: B**

**Explanation:** S3 Object Lock in compliance mode enforces a fixed retention period (set here to 365 days, satisfying the 1-year minimum) during which absolutely no user — including the AWS account root user — can delete or overwrite an object, guaranteeing the "no users can modify or delete" requirement in a way that is enforced by S3 itself rather than by IAM permissions that could be changed.

- A. Wrong — Governance mode allows users with special permissions (s3:BypassGovernanceRetention) to override the lock and delete or modify objects, so it does not guarantee that "no users" can modify or delete files; a legal hold also has no fixed duration, so it doesn't cleanly express a "minimum of 1 year" retention.
- C. Wrong — Restricting deletion/modification purely through IAM roles and bucket policies is a permissions-based control that a sufficiently privileged administrator (e.g., someone who can edit IAM policies) could change or remove, unlike Object Lock's compliance-mode guarantee that even the root user cannot bypass.
- D. Wrong — Tracking object hashes via Lambda only detects that a modification happened after the fact; it doesn't prevent modification or deletion at all, so it fails the "no users can modify or delete" requirement entirely.

## Question 71

A company hosts a website analytics application on a single Amazon EC2 On-Demand Instance. The analytics application is highly resilient and is designed to run in stateless mode.

The company notices that the application is showing signs of performance degradation during busy times and is presenting 5xx errors. The company needs to make the application scale seamlessly.

Which solution will meet these requirements MOST cost-effectively?

- A. Create an Amazon Machine Image (AMI) of the web application. Use the AMI to launch a second EC2 On-Demand Instance. Use an Application Load Balancer to distribute the load across the two EC2 instances.
- B. Create an Amazon Machine Image (AMI) of the web application. Use the AMI to launch a second EC2 On-Demand Instance. Use Amazon Route 53 weighted routing to distribute the load across the two EC2 instances.
- C. Create an AWS Lambda function to stop the EC2 instance and change the instance type. Create an Amazon CloudWatch alarm to invoke the Lambda function when CPU utilization is more than 75%.
- D. Create an Amazon Machine Image (AMI) of the web application. Apply the AMI to a launch template. Create an Auto Scaling group that includes the launch template. Configure the launch template to use a Spot Fleet. Attach an Application Load Balancer to the Auto Scaling group.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------+
 | Application Load       | --> | Auto Scaling group        | --> | Spot Fleet       |
 | Balancer                |     | (launch template + AMI,   |     | instances        |
 +------------------------+     |  scales seamlessly with    |     | (stateless,      |
                                 |  demand)                    |     |  resilient app)  |
                                 +----------------------------+     +------------------+
```

**Correct Answer: D**

**Explanation:** An Auto Scaling group built from a launch template (based on the AMI) scales the number of instances automatically and seamlessly in response to demand, directly fixing the performance degradation seen during busy times — unlike a fixed two-instance setup. Since the application is stateless and highly resilient, it can safely run on a Spot Fleet, taking advantage of Spot's steep discount versus On-Demand pricing for the most cost-effective solution, with the Application Load Balancer distributing traffic across however many instances are currently running.

- A. Wrong — Manually launching a fixed second On-Demand instance behind an ALB caps capacity at two instances and requires manual intervention to scale further; it doesn't scale "seamlessly" with fluctuating demand the way an Auto Scaling group does.
- B. Wrong — Same fixed-capacity problem as option A, and Route 53 weighted routing only splits traffic by DNS-level weighting between two static endpoints — it has no awareness of load or health-based scaling like an ALB with Auto Scaling does.
- C. Wrong — Stopping the instance and changing its instance type causes downtime during the resize and still only ever runs a single instance, which doesn't provide seamless scaling or resilience against a single instance's capacity limits.

## Question 72

A company is developing a file-sharing application that will use an Amazon S3 bucket for storage. The company wants to serve all the files through an Amazon CloudFront distribution. The company does not want the files to be accessible through direct navigation to the S3 URL.

What should a solutions architect do to meet these requirements?

- A. Write individual policies for each S3 bucket to grant read permission for only CloudFront access.
- B. Create an IAM user. Grant the user read permission to objects in the S3 bucket. Assign the user to CloudFront.
- C. Write an S3 bucket policy that assigns the CloudFront distribution ID as the Principal and assigns the target S3 bucket as the Amazon Resource Name (ARN).
- D. Create an origin access identity (OAI). Assign the OAI to the CloudFront distribution. Configure the S3 bucket permissions so that only the OAI has read permission.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+
 | Users    | --> | Amazon CloudFront      | --> | Amazon S3 bucket       |
 |          |     | Distribution (OAI       |     | (bucket policy allows |
 |          |     |  attached)              |     |  read only for OAI)   |
 +----------+     +------------------------+     +------------------------+
        (direct S3 URL access -> denied, no permission)
```

**Correct Answer: D**

**Explanation:** An origin access identity is a special CloudFront user identity that can be assigned to a distribution and then granted read permission on the S3 bucket via the bucket policy. Once the bucket policy grants access only to the OAI, requests made directly to the S3 URL are denied, and only requests routed through the CloudFront distribution (which authenticates as the OAI) can retrieve objects — exactly matching the requirement.

- A. Wrong — There's no native S3 bucket policy construct that grants access "to CloudFront" in general; access must be scoped to a specific principal such as an OAI (or OAC), which this option doesn't describe.
- B. Wrong — CloudFront cannot be "assigned" an IAM user, and IAM users are meant for people/applications with long-term credentials, not for authenticating a CDN distribution to an S3 origin.
- C. Wrong — A CloudFront distribution ID is not a valid IAM principal that can be referenced directly in an S3 bucket policy's Principal element; the correct mechanism is to reference the origin access identity (or origin access control), not the distribution ID itself.

## Question 73

Organizers for a global event want to put daily reports online as static HTML pages. The pages are expected to generate millions of views from users around the world. The files are stored in an Amazon S3 bucket. A solutions architect has been asked to design an efficient and effective solution.

Which action should the solutions architect take to accomplish this?

- A. Generate presigned URLs for the files.
- B. Use cross-Region replication to all Regions.
- C. Use the geoproximity feature of Amazon Route 53.
- D. Use Amazon CloudFront with the S3 bucket as its origin.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+
 | Users    | --> | Amazon CloudFront      | --> | Amazon S3 bucket       |
 | (global, |     | (edge caching, serves  |     | (origin, static HTML  |
 |  millions|     |  millions of views      |     |  reports)             |
 |  of views)|     |  efficiently)           |     +------------------------+
 +----------+     +------------------------+
```

**Correct Answer: D**

**Explanation:** Amazon CloudFront caches the static HTML reports at edge locations around the world, so the millions of expected views are served from the nearest edge cache instead of hitting the S3 origin directly for every request — this is the standard, efficient, and cost-effective way to serve high-volume static content globally.

- A. Wrong — Presigned URLs are for granting temporary, restricted access to private objects; they don't provide caching or performance benefits for high-volume public content, and add unnecessary complexity for files meant to be broadly viewed.
- B. Wrong — Cross-Region replication duplicates objects into buckets in other Regions but doesn't itself route users to the nearest copy or cache content at the edge; it also multiplies storage cost and requires additional routing logic to be useful.
- C. Wrong — Route 53 geoproximity routing directs DNS queries to different endpoints based on location, but without a CDN in front of the origin(s), it doesn't provide edge caching, and would require the replication/multi-endpoint setup that CloudFront handles natively and more simply.

## Question 74 (Choose two)

A company wants to migrate an on-premises data center to AWS. The data center hosts an SFTP server that stores its data on an NFS-based file system. The server holds 200 GB of data that needs to be transferred. The server must be hosted on an Amazon EC2 instance that uses an Amazon Elastic File System (Amazon EFS) file system.

Which combination of steps should a solutions architect take to automate this task? (Choose two.)

- A. Launch the EC2 instance into the same Availability Zone as the EFS file system.
- B. Install an AWS DataSync agent in the on-premises data center.
- C. Create a secondary Amazon Elastic Block Store (Amazon EBS) volume on the EC2 instance for the data.
- D. Manually use an operating system copy command to push the data to the EC2 instance.
- E. Use AWS DataSync to create a suitable location configuration for the on-premises SFTP server.

**Architecture Diagram:**

```
 +------------------------+     +----------------------+     +------------------------+
 | On-premises SFTP       | --> | AWS DataSync agent   | --> | Amazon EFS             |
 | server (NFS-based,     |     | (deployed on-prem,    |     | file system            |
 |  200 GB data)          |     |  reads NFS location)  |     | (mounted by EC2)       |
 +------------------------+     +----------------------+     +------------------------+
```

**Correct Answers: B, E**

**Explanation:** AWS DataSync automates data transfer between on-premises NFS/SMB storage and AWS storage services like EFS. Installing the DataSync agent in the on-premises data center (B) lets it read from the local NFS-based file system, and configuring DataSync with a location configuration for the on-premises SFTP server's file system (E) defines the source so DataSync can automatically and efficiently copy the 200 GB of data into the EFS file system, with no manual scripting required.

- A. Wrong — Amazon EFS is a Regional, multi-AZ service accessed through mount targets in each Availability Zone; the EC2 instance does not need to be launched into the "same Availability Zone as the EFS file system" for this migration to work, and this step doesn't address automating the data transfer itself.
- C. Wrong — The requirement is to use an EFS file system, not EBS; adding a secondary EBS volume doesn't fulfill the stated target storage or help automate the transfer.
- D. Wrong — Manually running an OS copy command is a manual process, not an automated one, which conflicts directly with the requirement to "automate this task."

## Question 75

A company hosts a containerized web application on a fleet of on-premises servers that process incoming requests. The number of requests is growing quickly. The on-premises servers cannot handle the increased number of requests. The company wants to move the application to AWS with minimum code changes and minimum development effort.

Which solution will meet these requirements with the LEAST operational overhead?

- A. Use AWS Fargate on Amazon Elastic Container Service (Amazon ECS) to run the containerized web application with Service Auto Scaling. Use an Application Load Balancer to distribute the incoming requests.
- B. Use two Amazon EC2 instances to host the containerized web application. Use an Application Load Balancer to distribute the incoming requests.
- C. Use AWS Lambda with a new code that uses one of the supported languages. Create multiple Lambda functions to support the load. Use Amazon API Gateway as an entry point to the Lambda functions.
- D. Use a high performance computing (HPC) solution such as AWS ParallelCluster to establish an HPC cluster that can process the incoming requests at the appropriate scale.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------------+
 | Application Load       | --> | Amazon ECS (Fargate)      | --> | Containerized web app |
 | Balancer                |     | Service Auto Scaling       |     | (existing containers, |
 +------------------------+     +----------------------------+     |  no code changes)      |
                                                                     +------------------------+
```

**Correct Answer: A**

**Explanation:** The application is already containerized, so running it on Amazon ECS with the Fargate launch type lets the existing container images run unchanged — no code rewrite needed — while Fargate removes the need to provision or manage any underlying servers. Service Auto Scaling automatically adjusts the number of running tasks to handle the growing request volume, and the Application Load Balancer distributes traffic across them, together giving the least operational overhead of the options presented.

- B. Wrong — Hosting on a fixed pair of EC2 instances requires the company to provision, patch, and manage the underlying servers and manually scale capacity, which is more operational overhead than a serverless Fargate/ECS approach.
- C. Wrong — Rewriting the application as new Lambda functions in a supported language directly conflicts with the requirement for "minimum code changes and minimum development effort," since the existing containerized code would need to be re-architected into function handlers.
- D. Wrong — An HPC cluster like AWS ParallelCluster is designed for tightly coupled, compute-intensive scientific/technical workloads (e.g., simulations), not for scaling a containerized web application serving incoming HTTP requests, and it would require significant re-architecture and operational management.

## Question 76

As part of budget planning, management wants a report of AWS billed items listed by user. The data will be used to create department budgets. A solutions architect needs to determine the most efficient way to obtain this report information.

Which solution meets these requirements?

- A. Run a query with Amazon Athena to generate the report.
- B. Create a report in Cost Explorer and download the report.
- C. Access the bill details from the billing dashboard and download the bill.
- D. Modify a cost budget in AWS Budgets to alert with Amazon Simple Email Service (Amazon SES).

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------------+
 | AWS billing data        | --> | AWS Cost Explorer          | --> | Report grouped by user  |
 | (cost allocation tags)  |     | (create report, group by   |     | / tag, downloadable for |
 |                          |     |  tag/linked account/user)  |     | department budgeting    |
 +------------------------+     +----------------------------+     +------------------------+
```

**Correct Answer: B**

**Explanation:** AWS Cost Explorer lets you build a custom report that filters and groups billed costs (for example, by cost allocation tag representing user or department), then save and download that report — directly producing the "billed items listed by user" data needed for department budgeting, with no custom querying or infrastructure required.

- A. Wrong — Amazon Athena can query billing data (e.g., Cost and Usage Reports in S3) but requires setting up the CUR export, an S3 bucket, a Glue/Athena table, and writing SQL — significantly more setup effort than using Cost Explorer's built-in reporting.
- C. Wrong — The billing dashboard shows overall bill totals and details but doesn't provide a way to break down and group costs specifically by user for budgeting purposes the way Cost Explorer's custom reports do.
- D. Wrong — AWS Budgets with an SES alert notifies when spending crosses a threshold; it doesn't generate a report of billed items broken down by user, so it doesn't meet the stated reporting requirement.

## Question 77

A company hosts its web application on AWS using seven Amazon EC2 instances. The company requires that the IP addresses of all healthy EC2 instances be returned in response to DNS queries.

Which policy should be used to meet this requirement?

- A. Simple routing policy
- B. Latency routing policy
- C. Multivalue routing policy
- D. Geolocation routing policy

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+
 | DNS query| --> | Amazon Route 53        | --> | Up to 8 healthy EC2   |
 |          |     | Multivalue Answer       |     | instance IP addresses |
 |          |     | routing policy          |     | returned in response  |
 +----------+     +------------------------+     +------------------------+
```

**Correct Answer: C**

**Explanation:** A multivalue answer routing policy lets Route 53 return multiple IP addresses (up to eight) in response to a single DNS query, with each associated with a health check so that only currently healthy resources are returned — directly matching the requirement to return the IP addresses of all healthy EC2 instances.

- A. Wrong — Simple routing returns a single set of values with no health check-based filtering built into its basic form, and it's designed for a single resource rather than distributing/returning multiple healthy endpoints.
- B. Wrong — Latency routing returns the single Region-based record with the lowest latency for the user, not a set of all healthy instance IPs.
- D. Wrong — Geolocation routing selects a single record based on the geographic location of the requester, not a set of all currently healthy IP addresses.

## Question 78

A company runs analytics software on Amazon EC2 instances. The software accepts job requests from users to process data that has been uploaded to Amazon S3. Users report that some submitted data is not being processed. Amazon CloudWatch reveals that the EC2 instances have a consistent CPU utilization at or near 100%. The company wants to improve system performance and scale the system based on user load.

What should a solutions architect do to meet these requirements?

- A. Create a copy of the instance. Place all instances behind an Application Load Balancer.
- B. Create an S3 VPC endpoint for Amazon S3. Update the software to reference the endpoint.
- C. Stop the EC2 instances. Modify the instance type to one with a more powerful CPU and more memory. Restart the instances.
- D. Route incoming requests to Amazon Simple Queue Service (Amazon SQS). Configure an EC2 Auto Scaling group based on queue size. Update the software to read from the queue.

**Architecture Diagram:**

```
 +------------------+     +------------------------+     +----------------------------+
 | Job requests     | --> | Amazon SQS queue       | --> | EC2 Auto Scaling group   |
 | (users, S3 data) |     | (buffers job requests) |     | (scales on queue size,    |
 +------------------+     +------------------------+     |  reads jobs from queue)   |
                                                            +----------------------------+
```

**Correct Answer: D**

**Explanation:** Routing incoming job requests through an SQS queue decouples request submission from processing, so no requests are dropped even when instances are momentarily saturated — jobs simply wait in the queue. Configuring the EC2 Auto Scaling group to scale based on queue size lets the fleet grow and shrink in direct response to actual user load, which both fixes the "not being processed" data loss caused by 100% CPU saturation and satisfies the requirement to scale based on demand.

- A. Wrong — Placing instances behind an ALB assumes the workload is request/response HTTP traffic distributed evenly, but it doesn't address the root cause (jobs overwhelming fixed capacity) or provide scaling tied to actual job backlog the way an SQS-driven Auto Scaling group does.
- B. Wrong — An S3 VPC endpoint improves network path/privacy for S3 access but does nothing to address CPU saturation or provide scaling based on user load.
- C. Wrong — Resizing to a more powerful instance type increases capacity for a fixed, single instance but doesn't scale with variable user load and still risks the same saturation problem recurring as demand grows further; it also requires downtime to stop/resize/restart.

## Question 79

A solutions architect needs to design a highly available application consisting of web, application, and database tiers. HTTPS content delivery should be as close to the edge as possible, with the least delivery time.

Which solution meets these requirements and is MOST secure?

- A. Configure a public Application Load Balancer (ALB) with multiple redundant Amazon EC2 instances in public subnets. Configure Amazon CloudFront to deliver HTTPS content using the public ALB as the origin.
- B. Configure a public Application Load Balancer with multiple redundant Amazon EC2 instances in private subnets. Configure Amazon CloudFront to deliver HTTPS content using the EC2 instances as the origin.
- C. Configure a public Application Load Balancer (ALB) with multiple redundant Amazon EC2 instances in private subnets. Configure Amazon CloudFront to deliver HTTPS content using the public ALB as the origin.
- D. Configure a public Application Load Balancer with multiple redundant Amazon EC2 instances in public subnets. Configure Amazon CloudFront to deliver HTTPS content using the EC2 instances as the origin.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+     +------------------+
 | Users    | --> | Amazon CloudFront      | --> | Public ALB             | --> | EC2 instances    |
 |          |     | (HTTPS, edge delivery) |     | (origin, public subnet)|     | (private subnets)|
 +----------+     +------------------------+     +------------------------+     +------------------+
```

**Correct Answer: C**

**Explanation:** Placing the EC2 instances in private subnets removes their direct exposure to the internet, and routing all traffic through the public ALB (which CloudFront uses as its origin) keeps a single, controlled entry point into the application. CloudFront still delivers HTTPS content from edge locations close to users for the least delivery time, so this combination achieves both the edge-performance requirement and the most secure network posture of the four options.

- A. Wrong — Placing the EC2 instances in public subnets exposes them directly to the internet, which is less secure than keeping them in private subnets behind the ALB.
- B. Wrong — Using the EC2 instances directly as the CloudFront origin bypasses the load balancer entirely, losing load-balancing/health-check benefits, and is an atypical, harder-to-secure origin configuration compared to using the ALB as the origin.
- D. Wrong — This combines both weaknesses: EC2 instances directly exposed in public subnets, and CloudFront bypassing the ALB to origin directly from the instances, making it the least secure option.

## Question 80

A company has a multi-tier application that runs six front-end web servers in an Amazon EC2 Auto Scaling group in a single Availability Zone behind an Application Load Balancer (ALB). A solutions architect needs to modify the infrastructure to be highly available without modifying the application.

Which architecture should the solutions architect choose that provides high availability?

- A. Create an Auto Scaling group that uses three instances across each of two Regions.
- B. Modify the Auto Scaling group to use three instances across each of two Availability Zones.
- C. Create an Auto Scaling template that can be used to quickly create more instances in another Region.
- D. Change the ALB in front of the Amazon EC2 instances in a round-robin configuration to balance traffic to the web tier.

**Architecture Diagram:**

```
 +------------------------+
 | Application Load        |
 | Balancer                 |
 +------------------------+
        |              |
        v              v
 +--------------+  +--------------+
 | AZ-a: 3 EC2  |  | AZ-b: 3 EC2  |
 | instances    |  | instances    |
 | (Auto Scaling|  | (Auto Scaling|
 |  group)      |  |  group)      |
 +--------------+  +--------------+
```

**Correct Answer: B**

**Explanation:** Spreading the same six instances as three per Availability Zone within the existing Auto Scaling group (still behind the same ALB) removes the single-AZ point of failure and provides high availability, all without changing a single line of the application itself — the minimal, correct fix for the stated requirement.

- A. Wrong — Spanning multiple Regions introduces significant additional complexity (cross-Region networking, data replication, Region-aware routing) far beyond what's needed to fix a single-AZ availability gap, and isn't a simple "modify the infrastructure" change.
- C. Wrong — A launch template that could "quickly create more instances in another Region" is a manual, reactive process, not the automatic, continuously available multi-AZ architecture the requirement calls for.
- D. Wrong — An ALB already balances traffic across targets by default; there's no "round-robin configuration" change needed, and this option doesn't address the actual problem, which is that all instances sit in a single Availability Zone.

## Question 81

A company's website provides users with downloadable historical performance reports. The website needs a solution that will scale to meet the company's website demands globally. The solution should be cost-effective, limit the provisioning of infrastructure resources, and provide the fastest possible response time.

Which combination should a solutions architect recommend to meet these requirements?

- A. Amazon CloudFront and Amazon S3
- B. AWS Lambda and Amazon DynamoDB
- C. Application Load Balancer with Amazon EC2 Auto Scaling
- D. Amazon Route 53 with internal Application Load Balancers

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+
 | Users    | --> | Amazon CloudFront      | --> | Amazon S3 bucket       |
 | (global) |     | (edge caching, fastest |     | (downloadable reports, |
 |          |     |  response time)         |     |  no servers to manage)|
 +----------+     +------------------------+     +------------------------+
```

**Correct Answer: A**

**Explanation:** The reports are static downloadable files, so storing them in Amazon S3 requires no infrastructure provisioning at all, and fronting the bucket with Amazon CloudFront caches those files at edge locations worldwide, giving the fastest possible response time for a global audience — all at S3/CloudFront's pay-per-use pricing, which is the most cost-effective fit for this static-content use case.

- B. Wrong — Lambda and DynamoDB are suited for dynamic, compute/data-driven workloads; they add unnecessary complexity and cost for simply serving downloadable static report files.
- C. Wrong — An ALB with EC2 Auto Scaling still requires provisioning and managing EC2 infrastructure, which conflicts with the requirement to "limit the provisioning of infrastructure resources," and doesn't provide edge caching for global response time the way CloudFront does.
- D. Wrong — Route 53 with internal Application Load Balancers still requires running and managing EC2/container infrastructure behind those load balancers, and internal ALBs aren't even internet-facing, so this doesn't meet the global, low-provisioning, fast-response requirements.

## Question 82

A company uses 50 TB of data for reporting. The company wants to move this data from on premises to AWS. A custom application in the company's data center runs a weekly data transformation job. The company plans to pause the application until the data transfer is complete and needs to begin the transfer process as soon as possible.

The data center does not have any available network bandwidth for additional workloads. A solutions architect must transfer the data and must configure the transformation job to continue to run in the AWS Cloud.

Which solution will meet these requirements with the LEAST operational overhead?

- A. Use AWS DataSync to move the data. Create a custom transformation job by using AWS Glue.
- B. Order an AWS Snowcone device to move the data. Deploy the transformation application to the device.
- C. Order an AWS Snowball Edge Storage Optimized device. Copy the data to the device. Create a custom transformation job by using AWS Glue.
- D. Order an AWS Snowball Edge Storage Optimized device that includes Amazon EC2 compute. Copy the data to the device. Create a new EC2 instance on AWS to run the transformation application.

**Architecture Diagram:**

```
 +------------------+     ship device     +------------------------+     +------------------------+
 | On-premises data | ------------------> | AWS Snowball Edge      | --> | Amazon S3 (imported   |
 | (50 TB, no spare |                     | Storage Optimized      |     |  data)                 |
 |  bandwidth)      |                     +------------------------+     +------------------------+
                                                                                     |
                                                                                     v
                                                                          +------------------------+
                                                                          | AWS Glue                |
                                                                          | (managed transformation |
                                                                          |  job, runs in AWS)      |
                                                                          +------------------------+
```

**Correct Answer: C**

**Explanation:** With no spare network bandwidth, physically shipping the 50 TB on an AWS Snowball Edge Storage Optimized device is the fastest and only practical way to move that volume of data without a network transfer. Once the data lands in AWS (via S3), recreating the weekly transformation job as an AWS Glue job means the company runs it as a fully managed ETL service in the cloud going forward, with no servers to provision or manage — the least operational overhead of the options that actually satisfy both the offline-transfer and continue-in-AWS requirements.

- A. Wrong — AWS DataSync transfers data over the network, but the data center has no available bandwidth for additional workloads, so DataSync cannot be used here regardless of how convenient it would otherwise be.
- B. Wrong — AWS Snowcone has a much smaller capacity (up to about 8 TB) than the 50 TB that needs to move, and deploying the transformation application onto the device runs it on the Snowcone itself rather than "in the AWS Cloud" as required.
- D. Wrong — Ordering a Snowball Edge with EC2 compute and standing up a new EC2 instance to run the transformation application requires provisioning and managing EC2 infrastructure, which is more operational overhead than using the fully managed AWS Glue service.

## Question 83 (Choose two)

A company is preparing a new data platform that will ingest real-time streaming data from multiple sources. The company needs to transform the data before writing the data to Amazon S3. The company needs the ability to use SQL to query the transformed data.

Which solutions will meet these requirements? (Choose two.)

- A. Use Amazon Kinesis Data Streams to stream the data. Use Amazon Kinesis Data Analytics to transform the data. Use Amazon Kinesis Data Firehose to write the data to Amazon S3. Use Amazon Athena to query the transformed data from Amazon S3.
- B. Use Amazon Managed Streaming for Apache Kafka (Amazon MSK) to stream the data. Use AWS Glue to transform the data and to write the data to Amazon S3. Use Amazon Athena to query the transformed data from Amazon S3.
- C. Use AWS Database Migration Service (AWS DMS) to ingest the data. Use Amazon EMR to transform the data and to write the data to Amazon S3. Use Amazon Athena to query the transformed data from Amazon S3.
- D. Use Amazon Managed Streaming for Apache Kafka (Amazon MSK) to stream the data. Use Amazon Kinesis Data Analytics to transform the data and to write the data to Amazon S3. Use the Amazon RDS query editor to query the transformed data from Amazon S3.
- E. Use Amazon Kinesis Data Streams to stream the data. Use AWS Glue to transform the data. Use Amazon Kinesis Data Firehose to write the data to Amazon S3. Use the Amazon RDS query editor to query the transformed data from Amazon S3.

**Architecture Diagram:**

```
 +------------------+     +------------------------+     +------------------------+     +------------------+
 | Multiple sources | --> | Kinesis Data Streams / | --> | Transform (Kinesis     | --> | Amazon S3        |
 | (real-time data) |     | Amazon MSK              |     | Data Analytics / Glue) |     +------------------+
 +------------------+     +------------------------+     +------------------------+              |
                                                                                                    v
                                                                                          +------------------+
                                                                                          | Amazon Athena    |
                                                                                          | (SQL query)      |
                                                                                          +------------------+
```

**Correct Answers: A, B**

**Explanation:** Both options describe complete, coherent, fully managed streaming pipelines: (A) Kinesis Data Streams ingests the real-time data, Kinesis Data Analytics transforms it with SQL, Kinesis Data Firehose delivers the transformed output to S3, and Athena runs ad-hoc SQL queries against it. (B) Amazon MSK ingests the streaming data from multiple sources, AWS Glue (which supports streaming ETL jobs) transforms it and writes it to S3, and Athena again provides the SQL query layer. Both pipelines correctly end with Athena, a serverless SQL query engine built to query data directly in S3.

- C. Wrong — AWS DMS is designed to migrate/replicate data from source databases, not to ingest general real-time streaming data from multiple heterogeneous sources; it's the wrong ingestion tool for this scenario.
- D. Wrong — The Amazon RDS query editor runs SQL against an RDS database instance; it cannot query data stored as files in Amazon S3, so this pipeline fails at the final query step.
- E. Wrong — Same flaw as option D: the RDS query editor cannot be used to query transformed data sitting in an S3 bucket, since it only queries RDS databases, not S3.

## Question 84

A company needs to review its AWS Cloud deployment to ensure that its Amazon S3 buckets do not have unauthorized configuration changes.

What should a solutions architect do to accomplish this goal?

- A. Turn on AWS Config with the appropriate rules.
- B. Turn on AWS Trusted Advisor with the appropriate checks.
- C. Turn on Amazon Inspector with the appropriate assessment template.
- D. Turn on Amazon S3 server access logging. Configure Amazon EventBridge (Amazon CloudWatch Events).

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------------+
 | Amazon S3 bucket       | --> | AWS Config rule            | --> | Compliance finding /   |
 | (configuration change) |     | (e.g. s3-bucket-public-    |     | remediation trigger    |
 +------------------------+     |  read-prohibited)          |     +------------------------+
                                +----------------------------+
```

**Correct Answer: A**

**Explanation:** AWS Config continuously records the configuration state of resources like S3 buckets and evaluates them against rules (managed or custom), flagging any configuration drift or unauthorized change as noncompliant — directly matching the requirement to detect unauthorized S3 bucket configuration changes.

- B. Wrong — AWS Trusted Advisor provides best-practice checks across cost, performance, security, and fault tolerance at a point-in-time, but it isn't designed for continuous, rule-based tracking of configuration changes to specific resources over time.
- C. Wrong — Amazon Inspector performs automated vulnerability and security assessments of EC2 instances and container images; it doesn't monitor or evaluate S3 bucket configuration settings.
- D. Wrong — S3 server access logging records requests made to a bucket (who accessed what), not configuration changes to the bucket's settings, so it doesn't detect unauthorized configuration changes on its own.

## Question 85 (Choose two)

A company is designing a cloud communications platform that is driven by APIs. The application is hosted on Amazon EC2 instances behind a Network Load Balancer (NLB). The company uses Amazon API Gateway to provide external users with access to the application through APIs. The company wants to protect the platform against web exploits like SQL injection and also wants to detect and mitigate large, sophisticated DDoS attacks.

Which combination of solutions provides the MOST protection? (Choose two.)

- A. Use AWS WAF to protect the NLB.
- B. Use AWS Shield Advanced with the NLB.
- C. Use AWS WAF to protect Amazon API Gateway.
- D. Use Amazon GuardDuty with AWS Shield Standard.
- E. Use AWS Shield Standard with Amazon API Gateway.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +----------------------------+
 | External | --> | Amazon API Gateway     | --> | Network Load Balancer      |
 | users    |     | (AWS WAF: blocks SQLi/ |     | (AWS Shield Advanced:      |
 |          |     |  XSS web exploits)     |     |  detects/mitigates large,  |
 +----------+     +------------------------+     |  sophisticated DDoS)       |
                                                   +----------------------------+
```

**Correct Answers: B, C**

**Explanation:** AWS WAF attaches to Amazon API Gateway (not to a Network Load Balancer, since WAF operates at Layer 7 and NLB is a Layer 4 service) and can inspect API requests for web exploits like SQL injection, satisfying that requirement. AWS Shield Advanced provides enhanced, automated detection and mitigation of large and sophisticated DDoS attacks and can protect resources such as an NLB (via its associated Elastic IP addresses), directly satisfying the DDoS requirement. Together these cover both stated threats at the correct layer for each resource.

- A. Wrong — AWS WAF cannot be attached to a Network Load Balancer; WAF web ACLs only support Layer 7 resources like API Gateway, CloudFront, ALB, and AppSync, not NLBs.
- D. Wrong — Amazon GuardDuty is a threat-detection service that generates findings, not a DDoS mitigation tool, and AWS Shield Standard only provides basic, automatic DDoS protection that isn't sufficient for large, sophisticated attacks — that level of protection requires Shield Advanced.
- E. Wrong — AWS Shield Standard again provides only baseline DDoS protection, not the advanced detection/mitigation needed for large, sophisticated attacks, and this option omits any web exploit (SQL injection) protection entirely.

## Question 86

A company has a serverless website with millions of objects in an Amazon S3 bucket. The company uses the S3 bucket as the origin for an Amazon CloudFront distribution. The company did not set encryption on the S3 bucket before the objects were loaded. A solutions architect needs to enable encryption for all existing objects and for all objects that are added to the S3 bucket in the future.

Which solution will meet these requirements with the LEAST amount of effort?

- A. Create a new S3 bucket. Turn on the default encryption settings for the new S3 bucket. Download all existing objects to temporary local storage. Upload the objects to the new S3 bucket.
- B. Turn on the default encryption settings for the S3 bucket. Use the S3 Inventory feature to create a .csv file that lists the unencrypted objects. Run an S3 Batch Operations job that uses the copy command to encrypt those objects.
- C. Create a new encryption key by using AWS Key Management Service (AWS KMS). Change the settings on the S3 bucket to use server-side encryption with AWS KMS managed encryption keys (SSE-KMS). Turn on versioning for the S3 bucket.
- D. Navigate to Amazon S3 in the AWS Management Console. Browse the S3 bucket's objects. Sort by the encryption field. Select each unencrypted object. Use the Modify button to apply default encryption settings to every unencrypted object in the S3 bucket.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------------+
 | S3 bucket               | --> | S3 Inventory report        | --> | S3 Batch Operations    |
 | (default encryption ON  |     | (.csv of unencrypted        |     | job (copy command      |
 |  for new objects)        |     |  existing objects)          |     |  re-encrypts existing  |
 +------------------------+     +----------------------------+     |  objects at scale)     |
                                                                     +------------------------+
```

**Correct Answer: B**

**Explanation:** Turning on default encryption ensures every future object is automatically encrypted with no further action. For the millions of existing unencrypted objects, S3 Inventory generates a list of them, and an S3 Batch Operations job can then run a copy-in-place operation across all of them at scale (re-writing each object with encryption applied) — all without manually touching individual objects or standing up a new bucket, making this the lowest-effort solution.

- A. Wrong — Manually downloading millions of objects to local storage and re-uploading them to a new bucket is extremely effort-intensive and slow, and also requires updating the CloudFront origin to point at the new bucket.
- C. Wrong — Changing the bucket's default encryption to SSE-KMS (and turning on versioning) only affects newly written objects going forward; it does nothing to encrypt the millions of objects that already exist unencrypted in the bucket.
- D. Wrong — Manually selecting and modifying encryption settings for each unencrypted object one at a time is completely impractical at a scale of millions of objects and is far more effort than a Batch Operations job.

## Question 87

A company runs a web-based portal that provides users with global breaking news, local alerts, and weather updates. The portal delivers each user a personalized view by using a mixture of static and dynamic content. Content is served over HTTPS through an API server running on an Amazon EC2 instance behind an Application Load Balancer (ALB). The company wants the portal to provide this content to its users across the world as quickly as possible.

How should a solutions architect design the application to ensure the LEAST amount of latency for all users?

- A. Deploy the application stack in a single AWS Region. Use Amazon CloudFront to serve all static and dynamic content by specifying the ALB as an origin.
- B. Deploy the application stack in two AWS Regions. Use an Amazon Route 53 latency routing policy to serve all content from the ALB in the closest Region.
- C. Deploy the application stack in a single AWS Region. Use Amazon CloudFront to serve the static content. Serve the dynamic content directly from the ALB.
- D. Deploy the application stack in two AWS Regions. Use an Amazon Route 53 geolocation routing policy to serve all content from the ALB in the closest Region.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+
 | Users    | --> | Amazon CloudFront      | --> | Application Load       |
 | (global) |     | (static content cached |     | Balancer (origin,      |
 |          |     |  at edge; dynamic       |     |  single Region)        |
 |          |     |  content routed over    |     +------------------------+
 |          |     |  CloudFront's optimized |
 |          |     |  network path)          |
 +----------+     +------------------------+
```

**Correct Answer: A**

**Explanation:** CloudFront can serve both static and dynamic content from a single origin: static content gets cached at edge locations for near-instant delivery, while requests for dynamic content still enter through the nearest CloudFront edge location and travel back to the origin over AWS's optimized backbone network rather than the public internet, reducing latency versus a direct connection — all without needing to deploy or synchronize the application stack across multiple Regions.

- B. Wrong — Deploying in two Regions with Route 53 latency routing helps route users to whichever Region is closer, but running only two Regions still leaves users far from both underserved, and it adds the operational complexity of keeping application stacks (and any state) synchronized across Regions — more effort than CloudFront's globally distributed edge network for the same or better latency benefit.
- C. Wrong — Serving dynamic content directly from the ALB (bypassing CloudFront) means every dynamic request travels the full public internet path from the user to the single Region, missing out on the latency benefit CloudFront provides even for non-cacheable content.
- D. Wrong — Same multi-Region operational complexity as option B, and Route 53 geolocation routing directs users based on geographic location rather than actual network latency, which can route a user to a Region that isn't actually the fastest for them.

## Question 88

A solutions architect is designing a VPC with public and private subnets. The VPC and subnets use IPv4 CIDR blocks. There is one public subnet and one private subnet in each of three Availability Zones (AZs) for high availability. An internet gateway is used to provide internet access for the public subnets. The private subnets require access to the internet to allow Amazon EC2 instances to download software updates.

What should the solutions architect do to enable Internet access for the private subnets?

- A. Create three NAT gateways, one for each public subnet in each AZ. Create a private route table for each AZ that forwards non-VPC traffic to the NAT gateway in its AZ.
- B. Create three NAT instances, one for each private subnet in each AZ. Create a private route table for each AZ that forwards non-VPC traffic to the NAT instance in its AZ.
- C. Create a second internet gateway on one of the private subnets. Update the route table for the private subnets that forward non-VPC traffic to the private internet gateway.
- D. Create an egress-only internet gateway on one of the public subnets. Update the route table for the private subnets that forward non-VPC traffic to the egress-only Internet gateway.

**Architecture Diagram:**

```
 +--------------------+     +--------------------+     +--------------------+
 | AZ-a: Public subnet |     | AZ-b: Public subnet |     | AZ-c: Public subnet |
 | + NAT Gateway        |     | + NAT Gateway        |     | + NAT Gateway        |
 +--------------------+     +--------------------+     +--------------------+
          |                          |                          |
          v                          v                          v
 +--------------------+     +--------------------+     +--------------------+
 | AZ-a: Private subnet|     | AZ-b: Private subnet|     | AZ-c: Private subnet|
 | (route: 0.0.0.0/0 ->|     | (route: 0.0.0.0/0 ->|     | (route: 0.0.0.0/0 ->|
 |  NAT GW in AZ-a)    |     |  NAT GW in AZ-b)    |     |  NAT GW in AZ-c)    |
 +--------------------+     +--------------------+     +--------------------+
```

**Correct Answer: A**

**Explanation:** A NAT gateway must be placed in a public subnet (so it can reach the internet gateway) and provides outbound-only internet access for instances in private subnets. Creating one NAT gateway per AZ, each with its own per-AZ private route table pointing to the NAT gateway in that same AZ, preserves high availability — if one AZ's NAT gateway or the AZ itself fails, the other AZs' private subnets are unaffected, and traffic never needs to cross AZ boundaries.

- B. Wrong — NAT instances (and NAT gateways) must be placed in a public subnet to route traffic to the internet gateway; placing them in the private subnets as described here would not give them the internet path needed to function as NAT devices.
- C. Wrong — A VPC can only have one internet gateway attached at a time, and attaching an internet gateway route directly to private subnets would make them public by definition, defeating the purpose of keeping them private.
- D. Wrong — An egress-only internet gateway only works with IPv6 traffic; since this VPC and its subnets use IPv4 CIDR blocks, an egress-only internet gateway cannot provide the required IPv4 internet access.

## Question 89

A company has an application that runs on Amazon EC2 instances and uses an Amazon Aurora database. The EC2 instances connect to the database by using user names and passwords that are stored locally in a file. The company wants to minimize the operational overhead of credential management.

What should a solutions architect do to accomplish this goal?

- A. Use AWS Secrets Manager. Turn on automatic rotation.
- B. Use AWS Systems Manager Parameter Store. Turn on automatic rotation.
- C. Create an Amazon S3 bucket to store objects that are encrypted with an AWS Key Management Service (AWS KMS) encryption key. Migrate the credential file to the S3 bucket. Point the application to the S3 bucket.
- D. Create an encrypted Amazon Elastic Block Store (Amazon EBS) volume for each EC2 instance. Attach the new EBS volume to each EC2 instance. Migrate the credential file to the new EBS volume. Point the application to the new EBS volume.

**Architecture Diagram:**

```
 +------------------------+     retrieve creds     +------------------------+
 | EC2 instances           | <--------------------- | AWS Secrets Manager    |
 | (application)           |                         | (auto-rotates DB      |
 +------------------------+                         |  credentials)          |
                                                       +------------------------+
                                                                  |
                                                                  v
                                                       +------------------------+
                                                       | Amazon Aurora DB       |
                                                       +------------------------+
```

**Correct Answer: A**

**Explanation:** AWS Secrets Manager natively stores database credentials and, with automatic rotation turned on, periodically rotates the Aurora database password on a schedule using a built-in Lambda rotation function — with no custom code or manual file management required, directly minimizing the operational overhead of credential management.

- B. Wrong — Systems Manager Parameter Store SecureString parameters can store credentials, but they don't have Secrets Manager's built-in automatic rotation integration for RDS/Aurora; achieving rotation would require significant custom automation.
- C. Wrong — Storing the credential file in an encrypted S3 bucket still requires the company to manually manage and rotate credentials in the file itself; it doesn't reduce credential management overhead at all, only changes where the file is stored.
- D. Wrong — Moving the credential file to an encrypted EBS volume is just a different storage location for the same locally-stored file; it doesn't add rotation or reduce the manual overhead of managing credentials.

## Question 90

A solutions architect is using Amazon S3 to design the storage architecture of a new digital media application. The media files must be resilient to the loss of an Availability Zone. Some files are accessed frequently while other files are rarely accessed in an unpredictable pattern. The solutions architect must minimize the costs of storing and retrieving the media files.

Which storage option meets these requirements?

- A. S3 Standard
- B. S3 Intelligent-Tiering
- C. S3 Standard-Infrequent Access (S3 Standard-IA)
- D. S3 One Zone-Infrequent Access (S3 One Zone-IA)

**Architecture Diagram:**

```
 +------------------+     +----------------------------+     +------------------------+
 | Media files      | --> | S3 Intelligent-Tiering      | --> | Multi-AZ resilient    |
 | (unpredictable   |     | (auto-moves objects between |     | storage, no retrieval |
 |  access pattern) |     |  frequent/infrequent tiers) |     | fees                   |
 +------------------+     +----------------------------+     +------------------------+
```

**Correct Answer: B**

**Explanation:** S3 Intelligent-Tiering automatically monitors access patterns and moves objects between a frequent-access tier and lower-cost infrequent-access tiers as needed, with no retrieval fees and no operational effort — a direct fit for files whose access pattern is unpredictable. It also stores data across multiple Availability Zones, satisfying the requirement to be resilient to the loss of an AZ, while minimizing cost for a mixed and unpredictable access pattern.

- A. Wrong — S3 Standard is resilient across multiple AZs but always charges frequent-access pricing, so it doesn't minimize costs for the files that are rarely accessed.
- C. Wrong — S3 Standard-IA is cheaper to store but charges a retrieval fee and is intended for data with a known, predictable infrequent-access pattern; applying it to frequently-accessed files or unpredictable access would increase costs due to retrieval fees.
- D. Wrong — S3 One Zone-IA stores data in only a single Availability Zone, so it is not resilient to the loss of an Availability Zone, directly violating that requirement.

## Question 91

A company has an ecommerce checkout workflow that writes an order to a database and calls a service to process the payment. Users are experiencing timeouts during the checkout process. When users resubmit the checkout form, multiple unique orders are created for the same desired transaction.

How should a solutions architect refactor this workflow to prevent the creation of multiple orders?

- A. Configure the web application to send an order message to Amazon Kinesis Data Firehose. Set the payment service to retrieve the message from Kinesis Data Firehose and process the order.
- B. Create a rule in AWS CloudTrail to invoke an AWS Lambda function based on the logged application path request. Use Lambda to query the database, call the payment service, and pass in the order information.
- C. Store the order in the database. Send a message that includes the order number to Amazon Simple Notification Service (Amazon SNS). Set the payment service to poll Amazon SNS, retrieve the message, and process the order.
- D. Store the order in the database. Send a message that includes the order number to an Amazon Simple Queue Service (Amazon SQS) FIFO queue. Set the payment service to retrieve the message and process the order. Delete the message from the queue.

**Architecture Diagram:**

```
 +------------------+     +------------------------+     +------------------------+
 | Checkout form    | --> | Database (order stored)| --> | SQS FIFO queue         |
 | (possible retry/ |     +------------------------+     | (message deduplication,|
 |  resubmit)       |                                     |  exactly-once delivery)|
 +------------------+                                     +------------------------+
                                                                      |
                                                                      v
                                                           +------------------------+
                                                           | Payment service        |
                                                           | (processes once,       |
                                                           |  deletes message)      |
                                                           +------------------------+
```

**Correct Answer: D**

**Explanation:** An SQS FIFO queue provides exactly-once processing and message deduplication (using the order number as a deduplication ID), so even if the checkout form is resubmitted and generates a duplicate message, the payment service processes the order only once. Combined with the payment service retrieving and then deleting the message after processing, this reliably prevents multiple orders for the same transaction.

- A. Wrong — Kinesis Data Firehose is a one-way delivery pipeline into destinations like S3/Redshift; it doesn't support consumers polling and processing individual messages with deduplication, so it can't prevent duplicate order processing.
- B. Wrong — Invoking a Lambda function from CloudTrail based on logged API activity is an indirect, poorly-suited mechanism for handling checkout submissions and has no built-in deduplication of resubmitted forms.
- C. Wrong — Standard SNS does not provide message deduplication or exactly-once delivery; a duplicate resubmission would still generate a duplicate notification and duplicate payment processing.

## Question 92

A research laboratory needs to process approximately 8 TB of data. The laboratory requires sub-millisecond latencies and a minimum throughput of 6 GBps for the storage subsystem. Hundreds of Amazon EC2 instances that run Amazon Linux will distribute and process the data.

Which solution will meet the performance requirements?

- A. Create an Amazon FSx for NetApp ONTAP file system. Set each volume's tiering policy to ALL. Import the raw data into the file system. Mount the file system on the EC2 instances.
- B. Create an Amazon S3 bucket to store the raw data. Create an Amazon FSx for Lustre file system that uses persistent SSD storage. Select the option to import data from and export data to Amazon S3. Mount the file system on the EC2 instances.
- C. Create an Amazon S3 bucket to store the raw data. Create an Amazon FSx for Lustre file system that uses persistent HDD storage. Select the option to import data from and export data to Amazon S3. Mount the file system on the EC2 instances.
- D. Create an Amazon FSx for NetApp ONTAP file system. Set each volume's tiering policy to NONE. Import the raw data into the file system. Mount the file system on the EC2 instances.

**Architecture Diagram:**

```
 +------------------+     import/export     +------------------------+     +------------------------+
 | Amazon S3 bucket | <--------------------> | Amazon FSx for Lustre  | --> | Hundreds of EC2        |
 | (raw data, 8 TB) |                        | (persistent SSD,        |     | instances (Amazon      |
 +------------------+                        |  sub-ms latency,        |     |  Linux, parallel        |
                                               |  6+ GBps throughput)    |     |  processing)            |
                                               +------------------------+     +------------------------+
```

**Correct Answer: B**

**Explanation:** Amazon FSx for Lustre is a high-performance parallel file system purpose-built for HPC/compute-intensive workloads, and its persistent SSD storage option delivers the sub-millisecond latencies and multi-GBps throughput this research workload requires. Linking it to S3 for import/export lets it stage the raw data efficiently and write results back, and it mounts natively on Linux EC2 instances for hundreds of instances to process data in parallel.

- A. Wrong — Amazon FSx for NetApp ONTAP is a general-purpose, feature-rich (snapshots, cloning, multi-protocol) file system, but it isn't the HPC-optimized choice for guaranteed sub-millisecond latency and 6+ GBps throughput that Lustre with SSD storage provides for this scale-out compute scenario.
- C. Wrong — Persistent HDD storage for FSx for Lustre has significantly higher latency than SSD and cannot reliably deliver sub-millisecond latencies, failing the stated performance requirement.
- D. Wrong — Same fundamental mismatch as option A: FSx for NetApp ONTAP is not the purpose-built HPC parallel file system that Lustre is, regardless of the tiering policy setting, so it isn't tuned to guarantee this level of throughput and latency.

## Question 93

A company is building a web-based application running on Amazon EC2 instances in multiple Availability Zones. The web application will provide access to a repository of text documents totaling about 900 TB in size. The company anticipates that the web application will experience periods of high demand. A solutions architect must ensure that the storage component for the text documents can scale to meet the demand of the application at all times. The company is concerned about the overall cost of the solution.

Which storage solution meets these requirements MOST cost-effectively?

- A. Amazon Elastic Block Store (Amazon EBS)
- B. Amazon Elastic File System (Amazon EFS)
- C. Amazon OpenSearch Service (Amazon Elasticsearch Service)
- D. Amazon S3

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+     +------------------------+
 | EC2 instances           | --> | Application Load Balancer  | --> | Amazon S3              |
 | (multiple AZs)          |     +----------------------------+     | (900 TB text documents,|
 +------------------------+                                        |  virtually unlimited,   |
                                                                     |  pay-per-use scaling)   |
                                                                     +------------------------+
```

**Correct Answer: D**

**Explanation:** Amazon S3 provides virtually unlimited, durable object storage with automatic scaling to handle any level of demand, and its pay-for-what-you-use pricing model makes it the most cost-effective way to store 900 TB of text documents accessed by EC2 instances across multiple Availability Zones — with no capacity to provision or manage ahead of time.

- A. Wrong — Amazon EBS volumes are attached to a single EC2 instance at a time and have a per-volume size limit far below 900 TB, and provisioning that much block storage would be significantly more expensive than S3 for this use case.
- B. Wrong — Amazon EFS can scale and be shared across instances/AZs, but its per-GB storage cost is substantially higher than S3, making it less cost-effective for storing 900 TB of documents.
- C. Wrong — Amazon OpenSearch Service is a search and analytics engine, not a general-purpose document storage solution, and running a cluster large enough to hold 900 TB would be far more expensive and operationally complex than storing files in S3.

## Question 94 (Choose two)

A solutions architect has created a new AWS account and must secure AWS account root user access.

Which combination of actions will accomplish this? (Choose two.)

- A. Ensure the root user uses a strong password.
- B. Enable multi-factor authentication to the root user.
- C. Store root user access keys in an encrypted Amazon S3 bucket.
- D. Add the root user to a group containing administrative permissions.
- E. Apply the required permissions to the root user with an inline policy document.

**Architecture Diagram:**

```
 +------------------------+     +----------------------------+
 | AWS account root user   | --> | Strong password             |
 |                          |     | + MFA device (virtual/     |
 |                          |     |   hardware token)           |
 +------------------------+     +----------------------------+
```

**Correct Answers: A, B**

**Explanation:** Best practice for securing the root user is to set a strong, unique password (A) and enable multi-factor authentication on that root account (B), so that even if the password is somehow exposed, an attacker still cannot sign in without the second factor. These are the two foundational root-user security controls AWS itself recommends, and neither requires creating or managing any additional credentials.

- C. Wrong — Best practice is to avoid creating root user access keys entirely; storing them (even encrypted) in S3 keeps long-lived, highly privileged credentials in circulation, which is a security risk rather than a mitigation.
- D. Wrong — The root user does not need to be (and should not be) added to an IAM group; day-to-day administrative work should be done through separate IAM users/roles with appropriate permissions, not the root user.
- E. Wrong — IAM policies (inline or otherwise) are not attached to the root user to grant permissions — the root user already has full, unrestricted access to the account by default, so this action doesn't apply or make sense.

## Question 95

A company is building a new dynamic ordering website. The company wants to minimize server maintenance and patching. The website must be highly available and must scale read and write capacity as quickly as possible to meet changes in user demand.

Which solution will meet these requirements?

- A. Host static content in Amazon S3. Host dynamic content by using Amazon API Gateway and AWS Lambda. Use Amazon DynamoDB with on-demand capacity for the database. Configure Amazon CloudFront to deliver the website content.
- B. Host static content in Amazon S3. Host dynamic content by using Amazon API Gateway and AWS Lambda. Use Amazon Aurora with Aurora Auto Scaling for the database. Configure Amazon CloudFront to deliver the website content.
- C. Host all the website content on Amazon EC2 instances. Create an Auto Scaling group to scale the EC2 instances. Use an Application Load Balancer to distribute traffic. Use Amazon DynamoDB with provisioned write capacity for the database.
- D. Host all the website content on Amazon EC2 instances. Create an Auto Scaling group to scale the EC2 instances. Use an Application Load Balancer to distribute traffic. Use Amazon Aurora with Aurora Auto Scaling for the database.

**Architecture Diagram:**

```
 +----------+     +------------------------+     +------------------------+
 | Users    | --> | Amazon CloudFront      | --> | Amazon S3 (static)     |
 |          |     |                         |     +------------------------+
 |          |     |                         | --> | API Gateway + Lambda   |
 |          |     |                         |     | (dynamic content)      |
 +----------+     +------------------------+     +------------------------+
                                                              |
                                                              v
                                                   +------------------------+
                                                   | DynamoDB (on-demand    |
                                                   | capacity, instant      |
                                                   | read/write scaling)    |
                                                   +------------------------+
```

**Correct Answer: A**

**Explanation:** This is a fully serverless stack: S3 and CloudFront serve static content with no servers to patch, API Gateway and Lambda run dynamic logic without managing any infrastructure, and DynamoDB with on-demand capacity scales both read and write throughput almost instantly in response to traffic changes with zero capacity planning — matching every stated requirement (no server maintenance, high availability, fastest possible read/write scaling).

- B. Wrong — Aurora, even with Aurora Auto Scaling, still runs on managed database instances that take time to add read replicas and does not scale write capacity as instantly as DynamoDB on-demand, which better matches the "as quickly as possible" requirement for both reads and writes.
- C. Wrong — Hosting content on EC2 instances (even in an Auto Scaling group) still requires patching and maintaining the underlying operating system, directly conflicting with the requirement to minimize server maintenance.
- D. Wrong — Same EC2 server maintenance problem as option C, and Aurora Auto Scaling still doesn't scale write capacity as quickly as DynamoDB on-demand capacity.

## Question 96

A reporting team receives files each day in an Amazon S3 bucket. The reporting team manually reviews and copies the files from this initial S3 bucket to an analysis S3 bucket each day at the same time to use with Amazon QuickSight. Additional teams are starting to send more files in larger sizes to the initial S3 bucket.

The reporting team wants to move the files automatically to the analysis S3 bucket as the files enter the initial S3 bucket. The reporting team also wants to use AWS Lambda functions to run pattern-matching code on the copied data. In addition, the reporting team wants to send the data files to a pipeline in Amazon SageMaker Pipelines.

What should a solutions architect do to meet these requirements with the LEAST operational overhead?

- A. Create a Lambda function to copy the files to the analysis S3 bucket. Create an S3 event notification for the analysis S3 bucket. Configure Lambda and SageMaker Pipelines as destinations of the event notification. Configure s3:ObjectCreated:Put as the event type.
- B. Create a Lambda function to copy the files to the analysis S3 bucket. Configure the analysis S3 bucket to send event notifications to Amazon EventBridge (Amazon CloudWatch Events). Configure an ObjectCreated rule in EventBridge (CloudWatch Events). Configure Lambda and SageMaker Pipelines as targets for the rule.
- C. Configure S3 replication between the S3 buckets. Create an S3 event notification for the analysis S3 bucket. Configure Lambda and SageMaker Pipelines as destinations of the event notification. Configure s3:ObjectCreated:Put as the event type.
- D. Configure S3 replication between the S3 buckets. Configure the analysis S3 bucket to send event notifications to Amazon EventBridge (Amazon CloudWatch Events). Configure an ObjectCreated rule in EventBridge (CloudWatch Events). Configure Lambda and SageMaker Pipelines as targets for the rule.

**Architecture Diagram:**

```
 +------------------+   S3 replication   +------------------------+     +------------------------+
 | Initial S3 bucket| ------------------> | Analysis S3 bucket     | --> | Amazon EventBridge     |
 +------------------+                      +------------------------+     | (ObjectCreated rule)   |
                                                                            +------------------------+
                                                                                |              |
                                                                                v              v
                                                                     +----------------+  +------------------+
                                                                     | AWS Lambda     |  | SageMaker        |
                                                                     | (pattern match)|  | Pipelines         |
                                                                     +----------------+  +------------------+
```

**Correct Answer: D**

**Explanation:** S3 replication automatically and natively copies new files from the initial bucket to the analysis bucket with no custom code to write or maintain, unlike a Lambda function that would have to be triggered and manage the copy itself. Native S3 event notifications only support a limited set of destinations (SQS, SNS, Lambda) and can't directly target SageMaker Pipelines, so routing the analysis bucket's ObjectCreated events through Amazon EventBridge lets a single rule fan out to both a Lambda function (for pattern-matching) and a SageMaker Pipelines target — meeting every requirement with fully managed, native AWS features and no custom copy logic.

- A. Wrong — Native S3 event notifications do not support SageMaker Pipelines as a destination type, so this configuration isn't achievable as described, and using a Lambda function to copy files adds unnecessary custom code compared to native S3 replication.
- B. Wrong — Using a Lambda function to copy files between buckets adds custom code and operational overhead that S3 replication handles natively and automatically.
- C. Wrong — Same limitation as option A: S3 event notifications can't directly target SageMaker Pipelines, so this configuration doesn't meet the requirement to send data to a SageMaker Pipelines pipeline.

## Question 97 (Choose two)

A social media company allows users to upload images to its website. The website runs on Amazon EC2 instances. During upload requests, the website resizes the images to a standard size and stores the resized images in Amazon S3. Users are experiencing slow upload requests to the website.

The company needs to reduce coupling within the application and improve website performance. A solutions architect must design the most operationally efficient process for image uploads.

Which combination of actions should the solutions architect take to meet these requirements? (Choose two.)

- A. Configure the application to upload images to S3 Glacier.
- B. Configure the web server to upload the original images to Amazon S3.
- C. Configure the application to upload images directly from each user's browser to Amazon S3 through the use of a presigned URL.
- D. Configure S3 Event Notifications to invoke an AWS Lambda function when an image is uploaded. Use the function to resize the image.
- E. Create an Amazon EventBridge (Amazon CloudWatch Events) rule that invokes an AWS Lambda function on a schedule to resize uploaded images.

**Architecture Diagram:**

```
 +----------+   presigned URL   +------------------------+     event     +------------------------+
 | User's   | ----------------> | Amazon S3               | ------------> | AWS Lambda             |
 | browser  |   direct upload   | (original image)         |  ObjectCreated| (resizes image, writes |
 +----------+                    +------------------------+               |  resized copy to S3)   |
                                                                            +------------------------+
```

**Correct Answers: C, D**

**Explanation:** Uploading directly from the user's browser to S3 using a presigned URL removes the EC2 web servers from the upload data path entirely, eliminating them as a bottleneck and decoupling the upload process from the application tier. Configuring an S3 event notification to invoke a Lambda function on each upload then handles resizing asynchronously and event-driven, further decoupling the resize step from the request/response cycle — together directly addressing both the slow uploads and the coupling problem with minimal operational overhead.

- A. Wrong — S3 Glacier is an archival storage class with long retrieval times; it isn't suitable for storing images that need to be actively served and resized as part of an interactive website.
- B. Wrong — Uploading the original images through the web server still routes all upload traffic through the EC2 instances, keeping the same bottleneck and coupling that's causing the slow uploads.
- E. Wrong — A scheduled Lambda invocation doesn't respond immediately to each new upload and would need to track/discover which images are new, making it far less efficient and more complex than a direct S3 event notification trigger.

## Question 98

A company is developing an application in the AWS Cloud. The application's HTTP API contains critical information that is published in Amazon API Gateway. The critical information must be accessible from only a limited set of trusted IP addresses that belong to the company's internal network.

Which solution will meet these requirements?

- A. Set up an API Gateway private integration to restrict access to a predefined set of IP addresses.
- B. Create a resource policy for the API that denies access to any IP address that is not specifically allowed.
- C. Directly deploy the API in a private subnet. Create a network ACL. Set up rules to allow the traffic from specific IP addresses.
- D. Modify the security group that is attached to API Gateway to allow inbound traffic from only the trusted IP addresses.

**Architecture Diagram:**

```
 +------------------+     +----------------------------+     +------------------------+
 | Requests         | --> | Amazon API Gateway         | --> | HTTP API (critical    |
 | (source IP       |     | Resource Policy:            |     | information)           |
 |  checked)        |     | Deny * unless source IP in  |     +------------------------+
                          | trusted CIDR list           |
                          +----------------------------+
```

**Correct Answer: B**

**Explanation:** An API Gateway resource policy can explicitly deny access to all requests except those originating from a specified set of IP address ranges (using a condition on the source IP), directly enforcing that only the company's trusted internal IP addresses can reach the API — a native, managed control built exactly for this purpose.

- A. Wrong — Private integration connects API Gateway to resources inside a VPC (such as an NLB) via a VPC link; it isn't a mechanism for restricting which client IP addresses can call the API.
- C. Wrong — API Gateway is a fully managed service that isn't deployed into a subnet you control, and network ACLs apply to VPC subnet traffic, not to a managed API Gateway endpoint, so this approach doesn't apply here.
- D. Wrong — Security groups attach to VPC resources with elastic network interfaces (like EC2 instances or NLBs); a regional/edge-optimized API Gateway endpoint doesn't have a security group that can be modified this way.

## Question 99

A company needs to migrate a legacy application from an on-premises data center to the AWS Cloud because of hardware capacity constraints. The application runs 24 hours a day, 7 days a week. The application's database storage continues to grow over time.

What should a solutions architect do to meet these requirements MOST cost-effectively?

- A. Migrate the application layer to Amazon EC2 Spot Instances. Migrate the data storage layer to Amazon S3.
- B. Migrate the application layer to Amazon EC2 Reserved Instances. Migrate the data storage layer to Amazon RDS On-Demand Instances.
- C. Migrate the application layer to Amazon EC2 Reserved Instances. Migrate the data storage layer to Amazon Aurora Reserved Instances.
- D. Migrate the application layer to Amazon EC2 On-Demand Instances. Migrate the data storage layer to Amazon RDS Reserved Instances.

**Architecture Diagram:**

```
 +------------------------+     +------------------------+
 | EC2 Reserved Instances | --> | Aurora Reserved         |
 | (application layer,    |     | Instances (database     |
 |  runs 24/7, steady)    |     |  layer, steady long-term|
 +------------------------+     |  commitment; storage    |
                                 |  auto-scales separately)|
                                 +------------------------+
```

**Correct Answer: C**

**Explanation:** Since the application runs continuously (24/7) with a steady, predictable long-term workload, committing to Reserved Instances for both the EC2 application layer and the Aurora database layer captures the deepest discount on compute versus On-Demand pricing. The fact that database storage keeps growing doesn't preclude using Aurora Reserved Instances: a Reserved Instance commitment covers the database's compute/instance class, while Aurora storage scales up automatically and is billed separately per GB, so growing storage doesn't require extra flexibility in the compute layer.

- A. Wrong — Spot Instances can be interrupted at any time, which is unsuitable for an application that must run continuously 24/7, and Amazon S3 is object storage, not a relational database storage layer suited to a legacy application's database.
- B. Wrong — Since the application (and by extension its database) runs continuously in a steady, predictable pattern, RDS On-Demand pricing leaves cost savings on the table compared to committing to Reserved Instances, which is cheaper for an always-on workload.
- D. Wrong — Running the application layer on-demand instead of Reserved Instances forgoes savings for a workload that is known to run 24/7 continuously, making this less cost-effective than reserving both layers.

## Question 100

A company has an application that places hundreds of .csv files into an Amazon S3 bucket every hour. The files are 1 GB in size. Each time a file is uploaded, the company needs to convert the file to Apache Parquet format and place the output file into an S3 bucket.

Which solution will meet these requirements with the LEAST operational overhead?

- A. Create an AWS Lambda function to download the .csv files, convert the files to Parquet format, and place the output files in an S3 bucket. Invoke the Lambda function for each S3 PUT event.
- B. Create an Apache Spark job to read the .csv files, convert the files to Parquet format, and place the output files in an S3 bucket. Create an AWS Lambda function for each S3 PUT event to invoke the Spark job.
- C. Create an AWS Glue table and an AWS Glue crawler for the S3 bucket where the application places the .csv files. Schedule an AWS Lambda function to periodically use Amazon Athena to query the AWS Glue table, convert the query results into Parquet format, and place the output files into an S3 bucket.
- D. Create an AWS Glue extract, transform, and load (ETL) job to convert the .csv files to Parquet format and place the output files into an S3 bucket. Create an AWS Lambda function for each S3 PUT event to invoke the ETL job.

**Architecture Diagram:**

```
 +------------------+     S3 PUT event     +------------------------+     +------------------------+
 | .csv file upload | ------------------->  | AWS Lambda (trigger)  | --> | AWS Glue ETL job       |
 | (hundreds/hour,  |                        +------------------------+     | (converts to Parquet,  |
 |  1 GB each)      |                                                       |  writes to S3)         |
 +------------------+                                                       +------------------------+
```

**Correct Answer: D**

**Explanation:** AWS Glue ETL jobs natively support converting data formats (including CSV to Parquet) as a fully managed, serverless service — no custom conversion code or cluster management is required. A lightweight Lambda function triggered on each S3 PUT event simply invokes the Glue job, so the heavy lifting (format conversion at scale for 1 GB files) is handled by Glue's managed Spark environment, giving the least operational overhead of the options.

- A. Wrong — Implementing custom CSV-to-Parquet conversion logic inside a Lambda function means the company must write, maintain, and test that conversion code itself, plus manage Lambda's memory/timeout limits for 1 GB files, which is more operational effort than using Glue's built-in format conversion.
- B. Wrong — Standing up and maintaining a custom Apache Spark job (and the infrastructure or service running it) adds significant operational overhead compared to a fully managed AWS Glue ETL job that already provides this capability out of the box.
- C. Wrong — Using a scheduled Lambda function to periodically query via Athena and convert results introduces polling delay (not truly event-driven per file) and requires custom code to convert Athena query results into Parquet, which is more complex and less immediate than a direct Glue ETL job triggered per upload.

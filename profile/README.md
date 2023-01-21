Checkout the website https://www.cloudanddevopsbabies.org/

There are two dynamic parts in the website - Testimonials and Blogs 

DynamoDB is used for storing both testimonials and blogs. DynamoDB streams can be used to enable event-driven programming by allowing to trigger specific actions in response to changes in the data in a DynamoDB table. They provide the ability to filter events using a filter expression. 

There are two filter expressions one for testimonials and another one for blogs

Testimonials can be added via the main website. Lambda function is called to insert item in DynamoDb. DynamoDB streams trigger the lambda function to replicate data to S3.Since the change in testimonials is not frequent, we can replicate to a JSON file in S3 bucket that is used as origin for cloud front.  This reduces the  hits to DynamoDB and the JSON file is cached at the edge.

Blogs are uploaded by Joe through admin page. It is uploaded to S3 and the meta data is inserted to DynamoDB via Lambda through S3 Event Notification. Similar to Testimonial , a JSON file is generated containing the list of blogs and the GitHub workflow action for the repo https://github.com/cdgbabies/main-website-frontend  is also invoked via Lambda . This repo corresponds to main page website which is built using Astro.  Every blog comes as a new page in the website . Since Astro expects all routes must be determined at build time (using SSG and not SSR), a dynamic route must export a `getStaticPaths()` that returns an array of objects with a `params` property. Each of these objects will generate a corresponding route. Blogs are stored in S3 bucket which is accessible via Origin Access Control via CloudFront.

Please find the process followed below:





![](https://screenshotsshiksha.s3.amazonaws.com/Flow.svg)



**Infrastructure Provisioning**

Terraform is used for provisioning resources. There are two repos 

https://github.com/cdgbabies/website-infra - Provisions entire infra needed for website

https://github.com/cdgbabies/lambda-code-s3-infra - S3 bucket for storing binaries corresponding to Lambda . It also creates role for GitHub action workflow since it is recommended to use role instead of accessing key pair for AWS credentials

**Compute**

Lambda is used for creating testimonial and event driven programming. 

https://github.com/cdgbabies/testimonials-ddb-update-handler-lambda - Triggered by DynamoDB stream when new testimonial is inserted to DynamoDB (Go)

https://github.com/cdgbabies/blogs-ddb-update-lambda - Triggered by DynamoDB stream when new metadata for blog is inserted to DynamoDB (Go)

https://github.com/cdgbabies/list-blogs-lambda - Triggered via function URL for listing blogs metadata stored in DynamoDB (Go)

https://github.com/cdgbabies/blogs-upload-handler-lambda - Triggered via S3 Event notification when new blog is uploaded to S3 (Java)

https://github.com/cdgbabies/add-testimonial-lambda - Triggered via function URL for inserting testimonial to DynamoDB (Java)

**Storage**

S3 is used for storing blogs. DynamoDB is used to store testimonials and blogs meta data. Single table design approach is used.

**Frontend**

https://github.com/cdgbabies/main-website-frontend - The website frontend is developed using Astro. Astro is a new framework for building content based websites. Checkout Astro website https://docs.astro.build/en/getting-started/ . Astro also enables us to include React components.  

https://github.com/cdgbabies/cdg-admin - There is also another page for Admin to upload blogs. In future, other features would be added. This is developed using React.

Tailwind CSS is used for styling

**CI/CD**

This is work in progress. GitHub Actions is used . Workflow is in place for deploying the static website. There are some workflows that are triggered manually to upload the build artifacts to S3 for lambdas. OpenID Connect (OIDC) allows GitHub Actions workflows to access resources in Amazon Web Services (AWS), without needing to store the AWS credentials as long-lived GitHub secrets. Terraform code provisions the OIDC provider and the roles used across the actions. 

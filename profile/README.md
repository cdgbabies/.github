
The website frontend is developed using Astro. Astro is a new framework for building content based websites. Checkout Astro website https://docs.astro.build/en/getting-started/ . Astro also enables us to include React components.  

There are two dynamic parts in the website - Testimonials and Blogs 

DynamoDB is used for storing both testimonials and blogs. DynamoDB streams can be used to enable event-driven programming by allowing to trigger specific actions in response to changes in the data in a DynamoDB table. They provide the ability to filter events using a filter expression. 

There are two filter expressions one for testimonials and another one for blogs

Testimonials can be added via the main website. Lambda function is called to insert item in DynamoDb. DynamoDB streams trigger the lambda function to replicate data to S3.Since the change in testimonials is not frequent, we can replicate to a JSON file in S3 bucket that is used as origin for cloud front.  This reduces the  hits to DynamoDB and the JSON file is cached at the edge.

Blogs are uploaded by Joe through admin page. It is uploaded to S3 and the meta data is inserted to DynamoDB via Lambda through S3 Event Notification. Similar to Testimonial , a JSON file is generated containing the list of blogs and the GitHub workflow action for the repo https://github.com/cdgbabies/main-website-frontend  is also invoked via Lambda . This repo corresponds to main page website which is built using Astro.  Every blog comes as a new page in the website . Since Astro expects all routes must be determined at build time (using SSG and not SSR), a dynamic route must export a `getStaticPaths()` that returns an array of objects with a `params` property. Each of these objects will generate a corresponding route. Blogs are stored in S3 bucket which is accessible via Origin Access Control via CloudFront.



![](https://screenshotsshiksha.s3.amazonaws.com/Untitled+Diagram.drawio+(1).svg)

---
title: "Analyze media content with AWS AI services"
date: 2025-06-02
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

Published: 2025-06-02 - Authors: Jack Bradham and Meera Balasubramaniam.

Organizations that manage large audio and video archives often struggle to extract value from their media libraries. Imagine a radio network with thousands of broadcast hours across multiple channels--they must verify ad placement, identify interview segments, and analyze programming patterns. In this post, we show how you can automatically convert unstructured media files into searchable, analyzable content. By combining Amazon Transcribe, Amazon Bedrock, Amazon QuickSight, and Amazon Q, organizations can achieve the following goals:

- Process and convert media into text as soon as files are uploaded
- Identify ad slots, interview segments, and topical programs
- Extract deeper insights with foundation models (FMs)
- Build a searchable knowledge base
- Deliver rich visualizations that drive decisions
- Enable natural-language queries across the entire media archive
- Visualize complex information with easy-to-understand graphics

In the sections that follow, you will see how these AWS services work together so your organization can tap into the full potential of its media, whether you need to ensure advertising compliance, analyze content, or locate specific moments within thousands of hours of recordings.

---

## Solution overview

This solution provides an event-driven media analytics pipeline that transforms how you manage and capture value from your content:

- **Optimize content management** - Automatically process media files on upload, saving time and reducing manual work.
- **Surface richer insights** - Produce accurate transcripts that capture context such as speaker, timing, and key moments.
- **Harness AI at scale** - Extract meaningful information and uncover hidden patterns without manually reviewing each file.
- **Build a searchable knowledge base** - Turn scattered media files into an explorable data hub for your entire team.
- **Create tailored interfaces** - Develop a customizable UI to search the data store.
- **Deliver powerful visualizations** - Convert findings into visuals that make even complex datasets easy to grasp.

The following diagram illustrates the architecture.

This event-driven architecture automatically processes and analyzes media using AWS services. The workflow includes the following steps:

1. Users upload media files into an Amazon Simple Storage Service (Amazon S3) bucket. The **New Media** event triggers the first AWS Step Functions workflow, which performs initial classification based on file-name values and launches transcription.
2. Amazon Transcribe converts audio to accurate, readable text. The transcripts are stored securely in an S3 bucket for further analysis, and the **Transcription Complete** event triggers the next phase.
3. A second Step Functions workflow processes the transcripts. Using predefined prompts, Amazon Bedrock analyzes the transcripts to extract actionable insights. The extracted metadata is saved into an S3 data lake.
4. The processed results are organized systematically, partitioned by date (year/month/day), and tagged with relevant attributes. This structured data enables natural-language querying through Amazon Q (when configured as a knowledge base), interactive visualizations through QuickSight, and seamless exploration.
5. Amazon Athena powers data exploration against the data lake. Athena acts as the data source for QuickSight, turning complex datasets into clear, compelling visuals.

This architecture transforms raw media into searchable, analyzable data while maintaining an organized structure for efficient access. The event-driven design ensures that new uploads are handled automatically, and the combination of AWS AI services unlocks a deeper understanding of the content. Each AWS service contributes to this transformation:

- **Amazon Bedrock** - Reviews transcripts to extract entities and insights: uses advanced foundation models, detects ads/interviews/programming segments, and derives actionable insights.
- **Amazon EventBridge** - Drives event-based orchestration: monitors new media and completed transcripts, and automatically triggers Step Functions workflows.
- **AWS Lambda** - Handles custom logic: bridges to Amazon Bedrock, executes tailored prompts, and adds flexible processing stages.
- **Amazon Q** - Acts as the user interface and Retrieval Augmented Generation (RAG) layer: provides enterprise-ready generative AI with built-in security controls such as SSO and responsible AI guardrails, enables natural-language queries across the archive, links responses to original media, and surfaces conversational access.
- **Amazon QuickSight** - Turns insights into compelling dashboards, visualizes media analytics end to end, and helps track ads, shows, and other patterns.
- **Amazon S3** - Stores original media, transcripts, and processed insights; automatically emits events when new content arrives.
- **AWS Step Functions** - Orchestrates intake and AI analysis phases, complete with robust error handling and retries.
- **Amazon Transcribe** - Converts speech to accurate text, identifies speakers, and adds timestamps for precise navigation.

---

## Security considerations

While this post focuses on the technical implementation of the media analytics pipeline, every production deployment must include comprehensive security controls.

**Secure storage in Amazon S3**

- Enable server-side encryption with AWS Key Management Service (AWS KMS) keys.
- Apply restrictive bucket policies so that only authorized principals can access data.
- Turn on Amazon S3 Block Public Access at the account and bucket levels.
- Enable versioning for recovery.
- Create lifecycle policies to manage retention windows.
- Enable S3 access logging.
- Use presigned URLs for temporary access.

**Identity and access management**

- Create least-privilege service roles for Step Functions, Amazon Transcribe jobs, Amazon Bedrock API calls, and Athena queries.
- Enforce role-based access control (RBAC).
- Rotate credentials regularly and enable multi-factor authentication (MFA).
- Use AWS Organizations to manage multi-account environments.

**Network security**

- Deploy VPC endpoints for Amazon S3, Athena, and QuickSight.
- Configure network ACLs and security groups aligned with your segmentation strategy.
- Enable VPC Flow Logs and use AWS PrivateLink where available.
- Configure route tables to control data flow.

**Data encryption**

- Apply AWS KMS encryption to S3 objects.
- Require TLS 1.2+ for every API interaction.
- Enable automatic key rotation and consider envelope encryption for sensitive data.

**Monitoring and detection**

- Enable AWS CloudTrail for API auditing.
- Turn on Amazon GuardDuty for threat detection.
- Configure Amazon CloudWatch metrics, alarms, and log groups for application telemetry.
- Enable S3 server access logging and VPC Flow Logs.

**Access control**

- Enforce fine-grained access for Amazon Bedrock models, Athena queries, and QuickSight dashboards.
- Review permissions regularly.

Compliance requirements and governance policies may affect how you implement this solution. Consult AWS security best practices and partner with your security team to tailor the controls to your use case. For more guidance, see **Best Practices for Security, Identity, & Compliance**.

---

## Prerequisites

To follow along, you need:

- An AWS account with an IAM user (or role) that can access Lambda, Amazon Bedrock, Amazon S3, Amazon Transcribe, Step Functions, and IAM.
- Access to the foundation models you plan to use in Amazon Bedrock. Refer to **Access Amazon Bedrock foundation models**.
- An Amazon QuickSight Enterprise subscription. See **Setting up for Amazon QuickSight**.

---

## Create the S3 buckets

This solution uses three distinct buckets to support the media analytics workflow:

- **Raw media bucket** - Stores the input files.
- **Transcription outputs bucket** - Stores Amazon Transcribe results.
- **Processed insights bucket** - Stores structured, AI-enriched data.

Follow the **Creating a general purpose bucket** guide to provision each bucket.

---

## Configure EventBridge

Enable event notifications on the raw input bucket so EventBridge can trigger automation. The system watches for activity in the S3 buckets--when new content is uploaded or transcripts are available, EventBridge invokes the right Step Functions workflow. For details, see **Creating rules that react to events in Amazon EventBridge**.

Example filter for newly uploaded media:

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["rawinputbucket"]
    }
  }
}
```

Example filter for newly added transcripts in the data lake:

```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["business-data-lake"]
    },
    "object": {
      "key": [{
        "suffix": ".transcription"
      }]
    }
  }
}
```

---

## Build the Step Functions workflows

The orchestration layer includes two primary workflows. The first handles media ingestion and transcription, while the second manages AI analysis. Each workflow includes safeguards for potential errors and retry logic. For a walkthrough, see **Learn how to get started with Step Functions**. One diagram illustrates how new media is ingested, indexed, and transcribed, and another diagram shows how transcripts are analyzed downstream.

---

## Set up Amazon Transcribe

Launch Amazon Transcribe jobs with the permissions you configured. Speech-to-text features such as language detection, speaker identification, and custom vocabularies improve accuracy for your media. Refer to **How Amazon Transcribe works** for setup details.

---

## Configure Amazon Bedrock

Drive the AI analyzer by crafting precise prompts that extract meaningful information. Amazon Bedrock examines transcripts to pinpoint segments, speakers, and topics, turning raw text into structured data. For guidance, see **Design a prompt**.

Example prompt:

```
You will be reviewing a radio transcription to identify advertisements and extract relevant details. Your task is to analyze the provided transcript and output the results in a specific JSON format based on a given schema.
Please follow these steps to complete the task:
1. Carefully read through the entire transcript.
2. Identify all advertisements within the transcript. Look for clear indicators such as product mentions, promotional language, or transitions from regular content to commercial content.
3. For each advertisement you identify, determine the following information:
    - Company: The name of the company being advertised
    - Start time: The timestamp in the transcript where the ad begins
    - End time: The timestamp in the transcript where the ad ends
    - Product: The specific product or service being advertised
4. Format your findings into a JSON object that follows the provided schema. Each advertisement should be a separate object within an array.
5. Ensure these fields in your response are provided for each advertisement.. All are required fields: company, starttime, endtime, product. 
6. Use precise timestamps for start and end times. If exact times are not available, make a best estimate based on the transcript's context.
7. If a particular field is unclear or not explicitly mentioned in the transcript, you may use "Unknown" as the value.
8. Only respond with json and nothing else. Do not provide comments or explain your answer. 
9. Surround the JSON response with standard ```json markers
Here's an example of how your output should be formatted:
{
"advertisements": [
        {
            "company": "TechGadgets Inc.",
            "starttime": "00:05:30",
            "endtime": "00:06:15",
            "product": "SmartHome Hub"
        },
        {
            "company": "FreshFoods Market",
            "starttime": "00:15:45",
            "endtime": "00:16:30",
            "product": "Organic Produce Delivery Service"
        }
    ]
}
Do not add any fields that are not specified in the schema, and ensure all required fields are present for each advertisement.
```

---

## Create a structured data lake

Adopt a hierarchical data-organization strategy to enable efficient storage and analytics. Use AWS Glue crawlers to discover and catalog media metadata automatically. For more details, see **Using crawlers to populate the Data Catalog**.

Configure Athena tables so you can run SQL-based queries on the media insights. Example view:

```sql
CREATE OR REPLACE VIEW "commercials_view" AS 
SELECT
  metadata.market market,
  metadata.station_call station_call,
  metadata.format_type format_type,
  CAST(metadata.timestamp AS timestamp) timestamp,
  ads.company adCompany,
  ads.product adProduct,
  ads.starttime,
  ads.endtime
FROM
  (commercials
CROSS JOIN UNNEST(advertisements) t (ads));
```

---

## Deploy Amazon Q

Enable natural-language interaction with the media archive using Amazon Q Business. Configure knowledge bases and metadata so users can search and access content through conversational queries. Use the processed insights bucket to populate the knowledge base. For setup, see **Getting started with Amazon Q Business**. The referenced screenshot shows example conversations with an AI assistant.

---

## Build QuickSight dashboards

With QuickSight, create visual analyses that make your insights tangible. Connect to Athena views to highlight ad trends, content analysis, and performance metrics through interactive dashboards. See **Tutorial: Create an Amazon QuickSight dashboard** for a complete walkthrough.

---

## Validate and optimize the media analytics solution

After you deploy the architecture, execute these activities to ensure the solution meets your performance and business goals.

**Establish comprehensive testing**

- Compare transcripts with original media.
- Verify the accuracy of AI-generated insights.
- Use representative samples from your library.

For example, select a recently processed radio show and compare its transcript with the original audio. Review the AI-generated insights to confirm that major events--such as ad transitions or interview segments--were detected correctly. To ensure the system handles all content types, sample a diverse mix, including morning talk shows, evening news, and weekend sports programming.

**Benchmark performance**

- Measure processing time across media types.
- Track AWS service resource usage.
- Identify bottlenecks in the workflow.

Monitor how long it takes to process different file lengths--from quick ads to full-length shows. Observe resource-consumption patterns to pinpoint bottlenecks, such as slower transcription for certain formats or opportunities to optimize parallelization.

**Validate real-world experience**

- Test natural-language queries with Amazon Q.
- Confirm the relevance of search results.
- Gather feedback from target users.

Have team members interact with Amazon Q using the questions they would normally ask, like "interviews about climate change last week." Collect feedback from distinct personas--content managers versus compliance reviewers--to refine the system.

This structured testing approach, combined with real-world scenarios, lays the groundwork for a robust, user-friendly media analytics solution.

As you transition from initial deployment to production, optimization becomes critical for both cost control and user satisfaction. A network that processes thousands of hours per week can unlock major savings and discovery improvements by refining transcription accuracy or throughput. Marketing teams analyzing ad placement also depend on precise insights. Consider the following optimization strategies:

- **Transcription refinement** - Adjust language models for domain-specific terminology, fine-tune speaker-diarization settings, and use custom vocabularies.
- **AI insight generation** - Iterate on prompts for more focused analysis, experiment with different models, and align extraction parameters with business objectives.
- **Scalability considerations** - Load-test with higher media volumes, configure auto scaling, and monitor the cost efficiency of the architecture.
- **Continuous improvement** - Establish periodic reviews, track key performance indicators (KPIs), and evolve the solution based on live usage metrics.

Start with a pilot deployment and expand gradually as you mature your media-analytics capabilities.

---

## Clean up resources

To avoid incurring ongoing charges, remove the resources you created after testing:

- **QuickSight** - Delete media-analytics dashboards and datasets, and deactivate the Enterprise subscription if it is no longer needed.
- **S3 buckets** - Empty and delete the raw media, transcription output, and processed insights buckets.
- **EventBridge rules** - Delete rules that monitor S3 bucket activity and remove their targets.
- **Step Functions workflows** - Delete the ingestion/transcription workflow and the AI-analysis workflow.
- **Lambda functions** - Remove functions that call Amazon Bedrock along with their IAM roles and policies.
- **Data lake components** - Delete Athena views and tables, AWS Glue crawlers and databases, and saved query results.
- **Amazon Q** - Delete the knowledge bases and any custom configurations.
- **Amazon Bedrock** - Remove custom prompts and deactivate FM access if you no longer need it.
- **Amazon Transcribe** - Delete custom vocabularies and stored transcription jobs.
- **IAM resources** - Delete custom roles and policies tied to this solution.
- **Additional clean-up** - Delete related CloudWatch log groups, alarms, or metrics, and remove saved Athena queries.

---

## Common use cases

Multiple industries can leverage this architecture to mine value from audio and video content. Tailor the solution for your needs, such as broadcast management, corporate communications, educational archives, and more.

- **Media and broadcasting** - Track advertising compliance, verify placement accuracy, and analyze programming at scale.
- **Corporate and enterprise** - Convert meeting recordings into searchable knowledge bases, identify key decisions and actions, and strengthen knowledge management.
- **Education and training** - Build comprehensive catalogs of course material, index training assets for quick lookup, and support continuous learning programs.
- **Legal services** - Produce timestamped transcripts, build searchable repositories of hearings, and streamline document review.
- **Healthcare** - Extract medical insights from consultations, categorize patient interactions, and assist clinical documentation.
- **Government and public sector** - Archive public meetings, auto-classify topics, and improve transparency and accessibility.
- **Customer service** - Analyze call-center recordings, identify service trends and pain points, and drive continuous experience improvements.

By harnessing AI, organizations can turn raw audio and video into structured insights that enable faster, better-informed decisions.

---

## Conclusion

This article demonstrates how to use AWS services to convert unstructured media into actionable intelligence. By combining Amazon Transcribe, Amazon Bedrock, QuickSight, and Amazon Q, you can build an automated, scalable media-analytics solution tailored to your organization's needs. The architecture provides:

- Automated processing of media files at scale
- AI-generated insights
- Natural-language search capabilities
- Decision-support visualizations
- Flexible, maintainable infrastructure

Organizations can now transform content into searchable knowledge, extract insights automatically, develop data-driven content strategies, and streamline operations through automation. As audio and video libraries continue to grow, the ability to process and extract value efficiently becomes increasingly vital. This architecture delivers a solid foundation for today's requirements while remaining adaptable to future innovations.

We invite you to explore how media analytics with AWS AI services can address your organization's unique challenges. Identify your top use cases and unlock the insights hidden across your media archives.

---

## About the authors

**Jack Bradham** is a Senior Solutions Architect at AWS with more than 20 years of leadership experience in technology. Before joining AWS, he held key roles at Microsoft and Google, advising federal agencies and Global 100 enterprises. Jack holds an MBA in International Business from the University of South Carolina and has deep expertise in cloud computing, enterprise architecture, and business transformation. He is passionate about helping customers design scalable cloud solutions that meet business goals through innovation.

**Meera Balasubramaniam** is a Senior Solutions Architect and Data Analytics Specialist at AWS. With over 20 years in technology, she helps organizations turn complex business challenges into actionable data solutions. Meera specializes in scalable data architectures, advanced analytics platforms, and cloud solutions. She brings extensive experience in enterprise data strategy, business intelligence, and machine learning, speaks frequently at industry events, and mentors the next generation of cloud experts.

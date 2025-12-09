---
title: "Improve conversational AI response times with Amazon Bedrock streaming API and AWS AppSync"
date: 2025-07-09
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

Published: 2025-07-09 - Authors: Salman Moghal and Philippe Duplessis-Guindon.

Many enterprises now use large language models (LLMs) in Amazon Bedrock to derive insights from internal data. Amazon Bedrock is a fully managed service that provides access to high-performing foundation models (FMs) from leading AI companies such as AI21 Labs, Anthropic, Cohere, Meta, Mistral AI, Stability AI, and Amazon through a single API, along with a rich set of tools to build generative AI applications with security, privacy, and responsible AI guardrails.

Organizations deploying conversational AI systems often face a common challenge: although APIs can quickly answer straightforward questions, complex queries that require reasoning and action (ReAct) chains take longer to process, which degrades the user experience. The issue is especially pronounced in regulated industries, where stringent security requirements raise the complexity even further. For example, a global financial-services organization with more than USD 1.5 trillion in assets under management faced this exact challenge. Even after successfully integrating multiple LLMs and data sources, they still needed a solution that honored strict security protocols (including Virtual Private Cloud, or VPC, boundaries and enterprise OAuth) while improving response time for complex prompts.

AWS AppSync is a fully managed service that lets developers build serverless GraphQL APIs with real-time capabilities. This post shows how to combine AWS AppSync subscriptions with Amazon Bedrock streaming endpoints to deliver incremental LLM responses. We provide an enterprise-ready implementation blueprint so regulated organizations can maintain compliance and boost user experience through instantaneous, real-time streaming responses.

---

## Solution overview

The solution uses AWS AppSync to launch an asynchronous conversational workflow. A core AWS Lambda function interacts with the Amazon Bedrock streaming API. As the LLM generates tokens, they are streamed to the frontend through AWS AppSync mutations and subscriptions.

A reference implementation of the Lambda function and AWS AppSync API is available in the accompanying sample code. The reference architecture diagram (described below) shows how the AWS services integrate to deliver the target outcome.

Here is how user requests are processed and how users receive real-time responses from an Amazon Bedrock LLM:

1. When a user loads the UI, the application subscribes to the GraphQL subscription `onSendMessage()`, which returns whether the WebSocket connection succeeded.
2. After the user enters a prompt, the application invokes the GraphQL query `getLLMResponse`, which triggers the data-source Lambda function.
3. The data-source Lambda publishes an event to an Amazon Simple Notification Service (Amazon SNS) topic, and a 201 response is returned to the user, completing the synchronous flow.
4. The sequence diagram illustrates these steps in detail.

Next, the Orchestrator Lambda is invoked by the published SNS event and starts the streaming flow by calling the Amazon Bedrock API `InvokeModelWithResponseStream`. Amazon Bedrock receives the user prompt, initiates a stream, and starts sending token chunks back to the Lambda function. Each time the Orchestrator Lambda receives a token chunk, it calls the GraphQL mutation `sendMessage`. That mutation triggers the `onSendMessage` subscription with partial LLM output, and the UI renders those tokens as soon as they arrive. A second sequence diagram breaks down this streaming phase step by step.

---

## Data and API design

The AppSync GraphQL schema includes three operation types: query, subscription, and mutation.

### Query operation

```graphql
input GetLlmResponseInput {
    sessionId: String!
    message: String!
    locale: String!
}

type Query {
    getLlmResponse(input: GetLlmResponseInput!): GetLlmResponse
        @aws_api_key
}
```

The synchronous `getLlmResponse` query accepts a unique `sessionId`, the user's `locale`, and the prompt `message`.

- The frontend must send a unique `sessionId` to identify the active chat session. The ID remains the same for the duration of the conversation, and a new ID is generated if the user reloads the UI.
- The `locale` indicates the user's language preference (for example `en_US`). Refer to **Languages and locales supported by Amazon Lex V2** for valid values.
- The `message` contains the user prompt that is passed to the LLM.

### Subscription operation

```graphql
type Subscription {
    onSendMessage(sessionId: String!): SendMessageResponse
        @aws_subscribe(mutations: ["sendMessage"])
        @aws_api_key
}
```

The `onSendMessage` subscription takes the `sessionId` and allows the frontend to open a WebSocket connection. This setup ensures the UI receives every response emitted by the `sendMessage` mutation for the same session.

### Mutation operation

```graphql
input SendMessageInput {
    sessionId: String!
    message: String!
    locale: String!
}

type Mutation {
    sendMessage(input: SendMessageInput!): SendMessageResponse
        @aws_api_key
        @aws_iam
}
```

The `sendMessage` mutation accepts a `SendMessageInput` payload. All required fields (denoted by `!`) must be supplied so that the message can be delivered to the frontend. The Orchestrator Lambda invokes this mutation to push partial LLM tokens to the UI; the next section covers this function.

---

## AWS AppSync data-source Lambda function

AWS AppSync invokes the data-source Lambda whenever the frontend calls the `getLlmResponse` query. This synchronous Lambda implementation, provided in the `bedrock-appsync-ds-lambda` GitHub repository, extracts the user prompt from the query, publishes it to an Amazon SNS topic, and returns a success status code to confirm the message reached the backend for processing.

---

## AWS AppSync Orchestrator Lambda function

The Orchestrator Lambda runs whenever an event is published to the SNS topic. It initializes the Amazon Bedrock streaming API by calling the Boto3 method `converse_stream`:

```python
brt = boto3.client(service_name="bedrock-runtime", region_name="us-west-2")
messages = []
message = {
    "role": "user",
    "content": [{"text": parsed_event["message"]}]
}
messages.append(message)
response = brt.converse_stream(
    modelId=model_id,
    messages=messages
)
```

The Lambda creates a Bedrock Runtime client, parses the SNS payload, and builds a prompt that follows the Amazon Bedrock Messages API (with `role` and `content`). The `modelId` is sourced from an environment variable, and the `messages` parameter must adhere to the Messages API schema. When `converse_stream` returns successfully, the Lambda iterates through the streaming body to send partial tokens to the frontend:

```python
stream = response.get("body")
if stream:
    self.appsync = AppSync(locale="en_US", session_id=session_id)
    self.appsync.invoke_mutation(DEFAULT_STREAM_START_TOKEN)
    event_count = 0
    buffer = ""
    for event in stream:
        if event:
            if list(event)[0] == "contentBlockDelta":
                event_count += 1
                buffer += event["contentBlockDelta"]["delta"]["text"]
            if event_count > 5:
                self.appsync.invoke_mutation(buffer)
                event_count = 0
                buffer = ""
    if len(buffer) != 0:
        self.appsync.invoke_mutation(buffer)
    self.appsync.invoke_mutation(DEFAULT_STREAM_END_TOKEN)
```

- As soon as the LLM begins responding, the Lambda sends a `DEFAULT_STREAM_START_TOKEN` through AWS AppSync to signal the UI to start rendering output.
- The Lambda buffers incoming chunks and only invokes the mutation after five chunks, which reduces network calls and latency while maintaining a fluid experience.
- After the final tokens are delivered, the Lambda emits `DEFAULT_STREAM_END_TOKEN` to tell the frontend the stream is complete.

See the `bedrock-orchestrator-lambda` repository for the full implementation.

---

## Prerequisites

To deploy the solution, install the Terraform CLI in your environment and complete the prerequisites listed in the companion GitHub documentation.

---

## Deploy the solution

1. Open a terminal window.
2. Navigate to the `deployment` directory.
3. Edit `sample.tfvars` and update the variables for your AWS environment:
   ```
   region = "us-west-2"
   lambda_s3_source_bucket_name = "YOUR_DEPLOYMENT_BUCKET"
   lambda_s3_source_bucket_key  = "PREFIX_WITHIN_THE_BUCKET"
   ```
4. Run the following commands:
   ```
   terraform init
   terraform apply -var-file="sample.tfvars"
   ```

Detailed deployment instructions are available in the **Deploy the solution** section of the GitHub repository.

---

## Test the solution

Use the provided sample web UI and run it inside VS Code (see the README for instructions). This UI subscribes to real-time responses and displays incremental tokens as they arrive from Amazon Bedrock.

---

## Clean up

Use the same `sample.tfvars` file from deployment to tear down the resources:

```
terraform destroy -var-file="sample.tfvars"
```

---

## Conclusion

This post demonstrated how to integrate the Amazon Bedrock streaming API with AWS AppSync subscriptions to dramatically improve conversational AI responsiveness and user satisfaction. By adopting streaming, the global financial-services organization cut initial response times for complex queries by roughly 75 percent (from 10 seconds to just 2-3 seconds), so users see answers as soon as they are generated instead of waiting for the full completion. The business impact is clear: lower abandonment rates, higher engagement, and a more responsive AI experience. You can deploy the provided Lambda code and Terraform templates to bring the same benefits to your environment quickly.

For additional flexibility, AWS AppSync Events offers an alternative deployment pattern that can further enhance real-time capabilities through a fully managed WebSocket API. By resolving the tension between comprehensive AI answers and speed, this streaming approach enables organizations to maintain high-quality interactions while delivering the responsiveness modern users expect.

---

## About the authors

**Salman Moghal** is a Principal Solutions Architect at AWS. He helps customers design secure, scalable data and AI solutions across regulated industries. Salman focuses on aligning technical architectures with business priorities and has deep experience with enterprise identity, VPC security, and generative AI platforms.

**Philippe Duplessis-Guindon** is a Senior Solutions Architect at AWS specializing in analytics and application integration. He partners with customers to launch real-time data applications and conversational AI experiences using services such as Amazon Bedrock, AWS AppSync, and AWS Lambda. Philippe is passionate about helping enterprises deliver responsive, compliant AI solutions.

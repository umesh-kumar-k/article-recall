---
aliases:
  - Multi Tenancy and Access Control
highlights: |-
  Access Control: Fine-grained permissions for tools/data per agent; integrate with enterprise IAM (LDAP,OAuth,SAML)

  Multi-Tenancy: Logical isolation via agent namespaces or physical isolation via dedicated instances; prevents cross-tenant data leakage
Source 1: https://clerk.com/blog/what-are-the-risks-and-challenges-of-multi-tenancy
Source 2: https://thenewstack.io/authorization-challenges-in-a-multitenant-system/
Source 3: https://auth0.com/blog/access-control-in-the-era-of-ai-agents/
Source 4: https://www.strongdm.com/blog/rbac-vs-abac
Source 5: https://www.pingidentity.com/en/resources/blog/post/policy-based-access-control.html
Source 6: https://permify.co/post/policy-based-access-control-pbac/
Source 7: https://workos.com/blog/ai-agent-access-control-best-practices
Source 8: https://aws.amazon.com/blogs/machine-learning/implementing-tenant-isolation-using-agents-for-amazon-bedrock-in-a-multi-tenant-environment/
Source 9:
---

# What are the risks and challenges of multi-tenancy?

## Risk #1: Data leakage or cross-tenant access
* **Key Points:**
  - The most serious multi-tenancy risk is when one tenant gains access to another tenant's data.
  - This is a direct violation of user trust and can lead to serious security breaches, regulatory penalties, and brand damage.
  - According to the OWASP Top 10 2021, broken access control (including cross-tenant access) is the most serious web application security risk.
  - IBM's Cost of a Data Breach Report 2025 found that the average cost of a data breach reached $4.45 million globally.
  - It's one of the quickest ways to lose enterprise customers.
  - Root cause: These issues often stem from missing or incorrect filters in backend queries: Failing to include the appropriate tenant in a WHERE clause; Using joins that don't scope by tenant, accidentally exposing data; APIs that don't enforce tenant context; Over-reliance on frontend-supplied data for authorization.
  - Mitigation: To implement proper authentication context in your APIs: Start by validating the session token and extracting the authenticated user's organization ID; Scope every database query with the organization context; Never rely on client-supplied organization data as technically inclined users could attempt to insert an alternate identifier to access unauthorized data; Verify the user has the required permissions for the requested action within that specific organization; Use claims embedded in the token or look them up in the database; Log all operations with both user and organization identifiers for audit trails.
  - This systematic approach creates multiple layers of protection against cross-tenant data access.
  - Tooling: Supabase's Row Level Security (RLS) is a powerful guardrail that enforces tenant-level access directly in the database. Modern authentication platforms strengthen this by embedding organization context in every session, ensuring tenant-aware access controls are enforced across the stack. Backend logic should always scope queries using the authenticated user's active organization as context.
* **Technical Entities (Classes/Functions/APIs):** `OWASP Top 10 2021`, `Supabase`, `Row Level Security (RLS)`
* **Code Snippet:** None

## Risk #2: Complex authorization logic
* **Key Points:**
  - Multi-tenant apps typically require that roles and permissions be scoped to the current organization. This complexity grows quickly when users belong to multiple organizations with different roles or permissions.
  - For example, a user might be authorized to read and write data in one organization, while only being able to read data in another. Mistakes in logic can lead to unauthorized access or unintentionally blocked functionality.
  - Without properly scoping roles to the active organization context, your app can easily misapply permissions. This leads to subtle, hard-to-debug issues.
  - Clerk simplifies this by maintaining role assignments per organization and allowing developers to embed those permissions (along with the organization identifier) directly in the token. That means your backend logic can consistently reference and trust those claims when determining access levels for a given user. This ensures that permissions are evaluated in the right context, reducing risk and simplifying logic.
* **Technical Entities (Classes/Functions/APIs):** `Clerk`
* **Code Snippet:** None

## Risk #3: Noisy neighbors
* **Key Points:**
  - Noisy neighbors occur when one tenant consumes a disproportionate amount of shared system resources. This can degrade performance for others and introduce instability. It's especially relevant in AI, analytics, or job-heavy workloads.
  - Root cause: Resource contention issues typically stem from: Shared queues and background tasks without tenant isolation; Unbounded API usage from individual customers; Lack of tenant-scoped rate limiting; One customer consuming too much CPU, memory, or database throughput.
  - Mitigation: Essential strategies to prevent noisy neighbor problems: Rate limit requests per organization_id using API gateway products; Partition job queues by tenant to prevent long-running jobs from blocking others; Monitor per-tenant usage metrics to proactively detect issues; Implement resource quotas and throttling at the tenant level; Set up alerts for unusual consumption patterns.
  - Tooling: Most modern API gateways offer built-in rate limiting by tenant identifier. This prevents one tenant from straining the API and causing timeouts for all users.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Risk #4: Compliance & isolation for regulated tenants
* **Key Points:**
  - Some industries—like healthcare, finance, and enterprise IT—require strong isolation, auditability, and control. These tenants often have compliance requirements like SOC 2, HIPAA, or internal vendor policies that require enhanced security and operational transparency.
  - Root cause: Multi-tenant architectures that treat all customers the same can fall short: Shared infrastructure may not meet data separation needs for regulated industries; Lack of custom domain support can be a procurement blocker; Missing audit trails prevent compliance verification; No data residency guarantees for sensitive workloads.
  - Mitigation: Strategies for regulatory compliance in multi-tenant environments: Implement enhanced isolation for regulated tenants; Provide detailed audit trails and operational transparency; Support custom domains for enterprise customers; Offer data residency controls where required; Maintain compliance certifications (SOC2, HIPAA, etc.).
  - Tooling: Clerk goes through regular auditing to ensure compliance with industry standards including SOC 2 Type II certification. Features like verified domains ensure that users signing in with work email addresses are mapped to the correct tenant by default.
  - For high-trust customers, consider hybrid approaches—dedicated database branches or projects which isolate that tenant's data onto their own Supabase instance. This further prevents data from leaking across tenants.
* **Technical Entities (Classes/Functions/APIs):** `SOC 2`, `HIPAA`, `Clerk`, `Supabase`
* **Code Snippet:** None

## Risk #5: Debugging and tenant context in logs
* **Key Points:**
  - Troubleshooting multi-tenant applications becomes harder when you can't easily map errors or behaviors to the right tenant. Without proper context in logs and telemetry, even simple bugs can become time-consuming to resolve.
  - Root cause: Common logging issues in multi-tenant apps: Apps log requests without properly tagging them with organization_id or user_id; Shared dev environments make it hard to impersonate or reproduce tenant-specific issues; Problems are compounded as the user base grows; Lack of contextual metadata makes debugging time-consuming.
  - Mitigation: Essential practices for effective multi-tenant debugging: Always include tenant and user metadata in logs, traces, and analytics events; Tag all operations with both user and organization identifiers; Implement user impersonation capabilities for support teams; Use structured logging with consistent metadata fields; Set up tenant-specific monitoring and alerting.
  - Tooling: Use observability tools like Sentry or PostHog that support custom metadata tags. Clerk easily integrates with these tools allowing you to tag which user is experiencing a particular issue and which organization they have active at that time. Additionally, we support impersonation so your support team can access your application in the context of that user, making it easier to experience the issue first-hand.
* **Technical Entities (Classes/Functions/APIs):** `Sentry`, `PostHog`, `Clerk`
* **Code Snippet:** None

## How modern tools help
* **Key Points:**
  - Clerk handles the heavy lifting of tenant-aware authentication. It supports per-organization roles, embeds organization context in sessions, and has a number of different features supporting enterprise readiness.
  - Supabase and Neon make it easier to enforce data isolation through Row Level Security. Neon (a serverless Postgres platform) supports per-organization database branches by default to isolate data as needed, whereas Supabase offers a full Backend as a Service which can be configured to support multitenancy or configured on a per-tenant basis using branches for flexibility.
  - Observability tools like Sentry and PostHog integrate tenant-level metadata so you can trace, debug, and optimize behavior per organization.
  - Designing with these risks in mind ensures your SaaS app is secure, scalable, and compliant—setting the stage for long-term success with enterprise-grade customers.
* **Technical Entities (Classes/Functions/APIs):** `Clerk`, `Supabase`, `Neon`, `Row Level Security`, `Sentry`, `PostHog`
* **Code Snippet:** None

## What's next?
* **Key Points:**
  - Ready to implement multi-tenancy in your app? Check out our practical guides: Implementing multi-tenancy into a Supabase app with Clerk - A step-by-step tutorial; How to Design a Multi-Tenant SaaS Architecture - Architecture patterns and best practices; B2B SaaS with Clerk - Learn about our complete B2B solution.
* **Technical Entities (Classes/Functions/APIs):** `Supabase`, `Clerk`
* **Code Snippet:** None

---

## Difference between RBAC vs. ABAC vs. ACL vs. PBAC vs. DAC

## Summary
* **Key Points:**
  - This article presents an overview of RBAC vs. ABAC, plus several additional models of access control, including PBAC, ACL, and DAC.
  - You will learn what these methods are, how they differ from each other, and the pros and cons of each.
  - This knowledge will help you choose the best-fit access control model for your organization so you can start on the path to implementation with confidence.
* **Technical Entities (Classes/Functions/APIs):** `RBAC`, `ABAC`, `PBAC`, `ACL`, `DAC`
* **Code Snippet:** None

## What is RBAC?
* **Key Points:**
  - Role-based access control (RBAC) is a security approach that authorizes and restricts system access to users based on their role(s) within an organization.
  - This method allows users to access the data and applications needed to fulfill their job requirements and minimizes the risk of unauthorized employees accessing sensitive information or performing unauthorized tasks.
  - In addition to restricting access, RBAC can define how a user interacts with data—permitting read-only or read/write access to certain roles, thus limiting a user's ability to execute commands or delete information.
  - Benefits of RBAC include: Security. RBAC uses the principle of least privilege to lower the risk of a data breach. It also limits damage should a breach occur. Ease of Use. RBAC connects employees to the data and systems they need and reduces administrative overhead for IT. Compliance Readiness. Administrators can more easily prove that data and sensitive information have been handled according to privacy, security, and confidentiality standards.
  - However, implementing RBAC requires collaboration across departments. Determining how to categorize roles and manage access for those roles requires a clear understanding of both the business and the technical infrastructure that supports it.
  - Assigning roles can be a challenge, especially as organizations grow and change. Admins may assign people to ill-fitted roles or add new roles contrary to policy, leading to privilege creep, role explosion, and security gaps.
  - ✨ Make it Easy: Implementing RBAC is arduous, but StrongDM makes it simple. Just define your roles directly in StrongDM or leverage roles in your identity provider (IdP). With your IdP, use StrongDM's SCIM integration to easily sync user and group permissions. Need to offboard an employee? The change in your IdP is reflected in StrongDM right away.
* **Technical Entities (Classes/Functions/APIs):** `RBAC`, `StrongDM`, `SCIM`
* **Code Snippet:** None

## What is ABAC?
* **Key Points:**
  - Organizations use attribute-based access control (ABAC) to achieve more fine-grained access control—either replacing or supplementing RBAC.
  - The difference between RBAC and ABAC stems from the way each method manages access. Unlike RBAC, which grants access according to predefined roles, ABAC is a security policy that relies on a combination of attributes to match users with the resources they need to do a job.
  - Attributes may consist of: user demographics include name, organization, job title, or security clearance. resource properties such as owner, creation date, or file type. environmental specifics such as time of day, location of access, and threat levels.
  - By evaluating attributes of both subjects and resources, ABAC allows for more flexibility in policy creation and enforcement. With ABAC, access is dynamic. Decisions can be made according to context and risk at runtime. ABAC policies can be enforced on servers, databases, clusters, and a host of other resources.
  - Key benefits of ABAC include: Granularity. Because it uses attributes rather than roles to specify relationships between users and resources, administrators can create precisely targeted rules without needing to create additional roles. Control. With ABAC, data teams can restrict access to personally identifiable information (PII) using tags. In the healthcare field, for example, administrators can connect tags to HIPAA privacy standards to automatically separate those who have access from those who don't. Flexibility. ABAC policies are easy to adapt as resources and users change. Rather than modifying rules or creating new roles, admins need only assign the relevant attributes to new users or resources. Adaptability. ABAC makes adding and revoking permissions easier by allowing admins to modify attributes. This simplifies onboarding and offboarding as well as the temporary provisioning of contractors and external partners. Security. ABAC allows admins to create context-sensitive rules as security needs arise so they can more easily protect user privacy and adhere to compliance requirements—without requiring a high degree of technical knowledge.
  - However, ABAC, whether stand-alone or as part of an RBAC and ABAC hybrid approach, takes time, effort, and resources to implement. Teams must define attributes, assign them to users and objects, and generate rules to govern the attributes and their application.
  - With both RBAC and ABAC, data teams have fine-grain access control, giving them even more security and productivity benefits. These controls can be assigned based on specific attributes, characteristics, and conditions to severely limit the viewing and use of sensitive data.
  - For example, a security administrator can limit users from accessing specific rows on a table that contain location data that is outside the user's region of responsibility. They can also mask data like social security numbers or employee salaries, or remove data access altogether outside specific working hours.
* **Technical Entities (Classes/Functions/APIs):** `ABAC`, `RBAC`
* **Code Snippet:** None

## What is PBAC?
* **Key Points:**
  - Policy-Based Access Control (PBAC) is another access management strategy that focuses on authorization.
  - Whereas RBAC restricts user access based on static roles, PBAC determines access privileges dynamically based on rules and policies.
  - Although PBAC is fairly similar to ABAC, ABAC requires more IT and development resources (e.g., XML coding) as the number of required attributes increases.
  - Some key benefits of PBAC include the following: Flexibility. Admins can generate policies that allow access to be fine- or coarse-grained. Speed. PBAC lets you quickly add, remove, or amend permissions. Adaptability. Policies address environmental and contextual controls (e.g., time- or location-bound access). Observability. Human-readable policies provide visibility into the relationship between identities and resources. Compliance. Align internal policies dynamically with access controls to ensure the organization is consistently in compliance.
  - These benefits may make policy-based access control more adaptable than some alternatives, especially as organizations grow and change. But every access control system will require some amount of management and thoughtful implementation. PBAC is no exception.
* **Technical Entities (Classes/Functions/APIs):** `PBAC`, `RBAC`, `ABAC`
* **Code Snippet:** None

## What is ACL?
* **Key Points:**
  - Access control lists (ACLs) control or restrict the flow of traffic through a digital environment.
  - ACL rules grant or deny access in two general categories: Filesystem ACLs apply to files and/or directories. The ACL specifies which subject (human user or machine/system process) is allowed access to objects and what operations are allowed on those objects. Networking ACLs apply to the network routers and switches. The ACL specifies which type of traffic can access the network and what activity is allowed.
  - Organizations use ACLs in tandem with VPNs to manage traffic. Doing so may improve network performance, increase security, and allow for more granular monitoring at entry and exit points.
  - This makes ACLs suitable for securing individual users and low-level data, but it may not be the best approach to access management in most business applications.
* **Technical Entities (Classes/Functions/APIs):** `ACL`, `VPN`
* **Code Snippet:** None

## What is DAC?
* **Key Points:**
  - Whereas RBAC, also known as non-discretionary access control, takes a more human-level approach to control access, Discretionary Access Control (DAC) uses ACLs to restrict access to resources, such as files and database tables, and define which privileges a user has for that resource.
  - With DAC, each user controls access to their own data. This method decentralizes security decisions, allowing resource owners to grant or restrict access as they see fit.
  - The primary benefits of DAC include the following: Simplicity. As long as a user and their privilege are correctly listed in an ACL, they can access the resource. Flexibility. Because access control is decentralized, resource owners may grant and revoke access to resources without going up the chain of command, allowing them to make decisions quickly. Granularity. The data owner can add or remove access based on individual needs.
  - For larger or more complex organizations, however, the simplicity and decentralization of DAC can also lead to problems. With DAC, security administrators may not have a clear mapping of how resources are accessed and interconnected throughout the network, particularly as changes to that access occur. This may lead to over-privileged users and conflicting permissions, which in turn compromise security.
* **Technical Entities (Classes/Functions/APIs):** `DAC`, `RBAC`, `ACL`
* **Code Snippet:** None

## Combine the Strengths of RBAC and ABAC
* **Key Points:**
  - Although ABAC is not necessarily a superset of RBAC—the two models can function independently—uniting attribute- and role-based access control helps DevOps teams achieve greater speed and flexibility, especially as environments become increasingly ephemeral.
  - StrongDM combines the strengths of RBAC and ABAC, allowing businesses to enforce either role- or attribute-based access controls.
  - Administrators can use static access rules (RBAC) to assign access to a specific resource(s) to a role. And they can use dynamic access rules (ABAC) to establish and enforce access rules based on granular attributes, including tags, resource types, and geographic location.
  - This fine-grain access control keeps security tight and data out of the wrong hands.
  - Access is granted dynamically, based on a resource's attributes, not the specific physical resource, so roles and users maintain just-right access every time a resource gets spun up or torn down. This flexibility is particularly useful for companies with lots of resources on the backend, especially ephemeral ones.
  - Benefits of StrongDM's dynamic access rules: Precise Control. Administrators can set conditional access controls based on any criteria they choose, make connections and relations between systems and users, and have granular, context-aware control over access to file system resources. Ease of use. Human words and simple sentences create access rules that dynamically change when resources change making access rules easier to enforce and quicker to implement. Time sensitivity. Administrators can temporarily approve elevated privileges for sensitive operations, reducing risk by putting time limits on higher levels of access. Better workflows. Dynamic access rules reduce administrative busywork, giving you more control when provisioning infrastructure and making it easier for staff to access the resources they need. Flexibility. By basing access rules on tags, StrongDM helps you keep pace with the ephemerality of today's computing landscape.
  - Managing access in the modern cloud presents new challenges for your organization. Ephemeral infrastructure and fast-paced production environments demand an access control model that balances accessibility with security.
  - RBAC and ABAC, as well as their alternatives, each, come with pros and cons, and understanding them will help you choose the right access control model for your organization. While any implementation will come with startup costs, the effort you put in today will yield great benefits in the long run.
* **Technical Entities (Classes/Functions/APIs):** `RBAC`, `ABAC`, `StrongDM`
* **Code Snippet:** None

---


## Implementing tenant isolation using Agents for Amazon Bedrock in a multi-tenant environment

## Architecture overview
* **Key Points:**
  - A tenant user signs in to an identity provider such as Amazon Cognito. They get a JSON Web Token (JWT), which they use for API requests. The JWT contains claims such as the user ID (or subject, sub), which identifies the tenant user, and the tenantId, which defines which tenant the user belongs to.
  - The tenant user inputs their question into the client application. The client application sends the question to a GraphQL API endpoint provided by AWS AppSync, in the form of a GraphQL mutation. You can learn more about this pattern in the blog post Build a Real-time, WebSockets API for Amazon Bedrock. The client application authenticates to AWS AppSync using the JWT from Amazon Cognito. The user is authorized using the Cognito User Pools integration.
  - The GraphQL mutation invokes using the EventBridge resolver. The event triggers an AWS Lambda function using an EventBridge rule.
  - The Lambda function calls the Amazon Bedrock InvokeAgent API. This function uses a tenant isolation policy to scope the permissions and generates tenant specific scoped credentials. More about this can be read in the blog Building a Multi-Tenant SaaS Solution Using AWS Serverless Services. Then, it sends the tenant ID, user ID and tenant specific scoped credentials to this API using the sessionAttributes parameter from the agent's sessionState.
  - The Amazon Bedrock agent determines what it needs to do to satisfy the user request by using the reasoning capabilities of the associated large language model (LLM). A variety of LLMs can be used, and for this solution we used Anthropic Claude 3 Sonnet. It passes the sessionAttributes object to an action group determined to help with the request, thereby securely forwarding tenant and user ID for further processing steps.
  - This Lambda function uses the provided tenant specific scoped credentials and tenant ID to fetch information from Amazon DynamoDB. Tenant configuration data is stored in a single, shared table, while user data is split in one table per tenant. After the correct data is fetched, it's returned to the agent. The agent interacts with the LLM for the second time to formulate a natural-language answer to the user based on the provided data.
  - The agent's response is published as another GraphQL mutation through AWS AppSync.
  - The client listens to the response using a GraphQL subscription. It renders the response to the user after it's received from the server.
  - Note that each component in this sample architecture can be changed to fit into your pre-existing architecture and knowledge in the organization. For example, you might choose to use a WebSocket implementation through Amazon API Gateway instead of using GraphQL or implement a synchronous request and response pattern. Whichever technology stack you choose to use, verify that you securely pass tenant and user context between its different layers. Do not rely on probabilistic components of your stack, such as an LLM, to accurately transmit security information.
* **Technical Entities (Classes/Functions/APIs):** `Amazon Cognito`, `JSON Web Token (JWT)`, `AWS AppSync`, `GraphQL`, `Amazon EventBridge`, `AWS Lambda`, `Amazon Bedrock InvokeAgent API`, `sessionAttributes`, `Anthropic Claude 3 Sonnet`, `Amazon DynamoDB`
* **Code Snippet:** None

## How tenant and user data is isolated
* **Key Points:**
  - This section describes how user and tenant data is isolated when a request is processed throughout the system.
  - Each step is discussed in more detail following the diagram.
  - For each prompt in the UI, the frontend sends the prompt as a mutation request to the AWS AppSync API and listens for the response through a subscription, as explained in step 8 of Figure 1 shown above.
  - The subscription is needed to receive the answer from the prompt, as the agent is invoked asynchronously.
  - Both the request and response are authenticated using Amazon Cognito, and the request's context, including user and tenant ID, is made available to downstream components.
* **Technical Entities (Classes/Functions/APIs):** `AWS AppSync`, `Amazon Cognito`
* **Code Snippet:**
```typescript
const answerId = user?.userId + "." + uuidv4();
```

```typescript
await client.graphql({
	authMode: "userPool",
    ...
    variables: {
      answerId,
      ...
    },
  });
```

```javascript
if (!ctx.args.answerId.startsWith(ctx.identity.sub + ".")) {
  util.unauthorized()
}
```

```python
tenant_id = event["detail"]["identity"]["claims"]["custom:tenantId"]
user_id = event["detail"]["identity"]["claims"]["sub"]
```

```typescript
statements: [
	new PolicyStatement({
		actions: ["dynamodb:Query"],
		resources: [tenantConfigurationTable.tableArn],
		conditions: {
			"ForAllValues:StringEquals": {
				"dynamodb:LeadingKeys": [
					"${aws:PrincipalTag/TenantId}"
				]}}}),
        new PolicyStatement({
actions: ["dynamodb:Query"], resources: ["arn:aws:dynamodb:*:*:table/${aws:PrincipalTag/TenantId}-orders"] }) ]
```

```python
response = client.invoke_agent(
    ...
sessionState={
"sessionAttributes": {
		"tenantId": tenant_id,
		"userId": user_id,
		"accessKeyId": credentials["accessKeyId"],
		"secretAccessKey":credentials["secretAccessKey"],
		"sessionToken": credentials["sessionToken"],
},)
```

```python
tenant_id = event["sessionAttributes"]["tenantId"]
user_id = event["sessionAttributes"]["userId"]
access_key_id = event["sessionAttributes"]["accessKeyId"]
secret_access_key = event["sessionAttributes"]["secretAccessKey"]
session_token = event["sessionAttributes"]["sessionToken"]

dynamodb = boto3.resource(
        "dynamodb",
        aws_access_key_id=event["sessionAttributes"]["accessKeyId"],
        aws_secret_access_key=event["sessionAttributes"]["secretAccessKey"],
        aws_session_token=event["sessionAttributes"]["sessionToken"],
    )
tenant_config_table_name = os.getenv("TENANT_CONFIG_TABLE_NAME")
tenant_config_table = dynamodb.Table(tenant_config_table_name)

orders_table_name = tenant_config_table.query(
    KeyConditionExpression=Key("tenantId").eq(tenant_id)
)["Items"][0]["ordersTableName"]
...
orders_table.query(KeyConditionExpression=Key("userId").eq(user_id))[
    "Items"
]
```

## Walkthrough
* **Key Points:**
  - In this section, you will set up the sample AI assistant described in the previous sections in your own AWS account.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

### Prerequisites
* **Key Points:**
  - For this walkthrough, you should have the following prerequisites: An AWS account with administrator access to the us-east-1 (North Virginia) AWS Region; AWS Cloud Development Kit (CDK) installed and configured on your machine; git client.
* **Technical Entities (Classes/Functions/APIs):** `AWS Cloud Development Kit (CDK)`
* **Code Snippet:** None

### Enable large language model
* **Key Points:**
  - An agent needs a large language model (LLM) to reason about the best way to fulfil a user request and formulate natural-language answers.
  - Follow the Amazon Bedrock model access documentation to enable Anthropic Claude 3 Sonnet model access in the us-east-1 (N. Virginia) Region.
  - After enabling the LLM, you will see the following screen with a status of Access granted.
* **Technical Entities (Classes/Functions/APIs):** `Anthropic Claude 3 Sonnet`
* **Code Snippet:** None

### Deploy sample application
* **Key Points:**
  - We prepared most of the sample application's infrastructure as an AWS Cloud Development Kit (AWS CDK) project.
  - If you have never used the CDK in the current account and Region (us-east-1), you must bootstrap the environment using the following command: `cdk bootstrap`
  - Using your local command line interface, issue the following commands to clone the project repository and deploy the CDK project to your AWS account.
  - This takes about 3 minutes, after which you should see output similar to the following.
  - In addition to the AWS resources shown in Figure1, this AWS CDK stack provisions three users, each for a separate tenant, into your AWS account.
  - Note down the passwords for the three users from the CDK output, labelled MultiTenantAiAssistantStack.tenantXPassword. You will need them in the next section. If you come back to this walkthrough later, you can retrieve these values from the file cdk/cdk-output.json generated by the CDK. Note that these are only initial passwords and need to be changed on first sign-in of each user.
  - You have now successfully deployed the stack called MultiTenantAiAssistantStack.
* **Technical Entities (Classes/Functions/APIs):** `AWS Cloud Development Kit (AWS CDK)`
* **Code Snippet:**
```bash
git clone https://github.com/aws-samples/multi-tenant-ai-assistant
cd multi-tenant-ai-assistant/cdk
npm install
cdk deploy 
cd ..
```

### Start the frontend and sign in
* **Key Points:**
  - Now that the backend is deployed and configured, you can start the frontend on your local machine, which is built in JavaScript using React. The frontend automatically pulls information from the AWS CDK output, so you don't need to configure it manually.
  - Issue the following commands to install dependencies and start the local webserver.
  - Open the frontend application by visiting localhost:3000 in your browser. You should see a sign-in page.
  - For Username, enter tenant1-user. For Password, enter the password you have previously retrieved from CDK output.
  - Set a new password for the user.
  - On the page Account recovery requires verified contact information, choose Skip.
  - You're now signed in and can start interacting with the agent.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```bash
cd frontend
npm install
npm run dev
```

### Interact with the agent
* **Key Points:**
  - You have completed the setup of the architecture shown in Figure 1 in your own environment. You can start exploring the web application by yourself or follow the steps suggested below.
  - Under Enter your Prompt, enter the following question logged in as tenant1-user: What is your return policy? You should receive a response that you can return items for 10 days. Tenant 2 has a return policy of 20 days, tenant 3 of 30 days.
  - Under Enter your Prompt, enter the following question: Which orders did I place? You should receive a response that you have not placed any orders yet.
  - You have now verified the functionality of the application. You can also try to access data from another user, and you will not get an answer due to the scoped IAM policy. For example, you can modify the agent and hardcode a tenant ID (such as tenant2). In the UI, sign in as the tenant1 user and you will see that with the generated tenant1 scoped credentials you will not be able to access tenant2 resources and you will get an AccessDeniedException. You can also see the error in the CloudWatch Logs for the AgentTask Lambda function.
* **Technical Entities (Classes/Functions/APIs):** `IAM`, `CloudWatch Logs`, `AgentTask Lambda`
* **Code Snippet:**
```
[ERROR] ClientError: An error occurred (AccessDeniedException) when calling the Query operation: User: *****/agentTaskLambda is not authorized to perform: dynamodb:Query on resource: TABLE  because no identity-based policy allows the dynamodb:Query action
```

### Add test data
* **Key Points:**
  - To simplify the process of adding orders to your database, we have written a bash script that inserts entries into the order tables.
  - In your CLI, from the repository root folder, issue this command to add an order for tenant1-user: ./manage-orders.sh tenant1-user add
  - Return to the web application and issue the following prompt: Which orders did I place? The agent should now respond with the order that you created.
  - Issue the following command to delete the orders for tenant1-user: ./manage-orders.sh tenant1-user clear
  - Repeat steps 1 through 3 with multiple orders. You can create a new user in Amazon Cognito and sign in to see that no data from other users can be accessed. The implementation is detailed in Figure 2.
* **Technical Entities (Classes/Functions/APIs):** `Amazon Cognito`
* **Code Snippet:** None

### Clean up
* **Key Points:**
  - To avoid incurring future charges, delete the resources created during this walkthrough. From the cdk folder of the repository, run the following command: cdk destroy
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:**
```bash
cdk destroy
```

## Conclusion
* **Key Points:**
  - Enabling secure multi-tenant capabilities in AI assistants is crucial for maintaining data privacy and preventing unauthorized access. By following the approach outlined in this blog post, you can create an AI assistant that isolates tenants while using the power of large language models.
  - The key points to remember are: When building multi-tenant SaaS applications, always enforce tenant isolation (leverage IAM where ever possible). Securely pass tenant and user context between deterministic components of your application, without relying on an AI model to handle this sensitive information. Use Agents for Amazon Bedrock to help build an AI assistant that can securely pass along tenant context. Implement isolation at different layers of your application to verify that users can only access data and resources associated with their respective tenant and user context.
  - By following these principles, you can build AI-powered applications that provide a personalized experience to users while maintaining strict isolation and security. As AI capabilities continue to advance, it's essential to design architectures that use these technologies responsibly and securely.
  - Remember, the sample application demonstrated in this blog post is just one way to approach multi-tenant AI assistants. Depending on your specific requirements, you might need to adapt the architecture or use different AWS services.
  - To continue learning about generative AI patterns on AWS, visit the AWS Machine Learning Blog. To explore SaaS on AWS, start by visiting our SaaS landing page. If you have any questions, you can start a new thread on AWS re:Post or reach out to AWS Support.
* **Technical Entities (Classes/Functions/APIs):** `IAM`, `Agents for Amazon Bedrock`
* **Code Snippet:** None



---
aliases:
  - RBAC RAG
---
# Authorizing access to data with RAG implementations

* **Key Points:**
  - Organizations are increasingly using large language models (LLMs) to provide new types of customer interactions through generative AI-powered chatbots, virtual assistants, and intelligent search capabilities.
  - To enhance these interactions, organizations are using Retrieval-Augmented Generation (RAG) to incorporate proprietary data, industry-specific knowledge, and internal documentation to provide more accurate, contextual responses.
  - With RAG, LLMs use an external knowledge base that uses a vector store to incorporate specific knowledge data before generating responses.
  - Our customers have told us that they're concerned adding additional context to prompts will lead to leakage of sensitive information to principals (persons or applications) that might exist in some of these tools or to unstructured data within the knowledge base.
  - As mentioned in previous posts (Part 1, Part 2), LLMs should be considered untrusted entities because they do not implement authorization as part of a response.
  - A good mental model for organizations is to assume that any data passed to an LLM as part of a prompt could be returned to the principal.
  - With tools (APIs that an LLM can invoke to interact with external resources), you can pass the identity tokens of the principal to the tool to determine what the principal is permitted to access and actions that are allowed.
  - Capabilities across different vector databases—including metadata filters and syncing identity information between the data source and the knowledge base—support providing better results from the knowledge base and provide a baseline filtering capability.
  - This does not provide for strong authorization capabilities using the data source as the source of truth, which some customers are looking for.
  - In this blog post, I show you an architecture pattern for providing strong authorization for results returned from knowledge bases with a walkthrough example of this using Amazon S3 Access Grants with Amazon Bedrock Knowledge Bases.
  - I also provide an outline of considerations when implementing similar architecture patterns with other data sources.
* **Technical Entities (Classes/Functions/APIs):** `RAG`, `LLMs`, `Amazon S3 Access Grants`, `Amazon Bedrock Knowledge Bases`, `IAM Identity Center`

## RAG usage overview
* **Key Points:**
  - RAG architectures share similarities with search engines but have key differences.
  - While both use indexed data sources to find relevant information, their approaches to data access differ.
  - Search engines provide links to information sources, requiring users to access the original data source directly based on their permissions.
  - Unlike search engines, RAG implementations return vector database results directly from the LLM, bypassing permission checks at the original data source.
  - While metadata filtering can help control access, it presents two key challenges. First, vector databases only sync periodically, meaning permission changes in the source data aren't immediately reflected. Second, complex identity permissions—where principals might belong to hundreds of groups—make it difficult to accurately filter results.
  - This makes metadata filtering insufficient for organizations that require stronger authorization controls.
  - To implement robust authorization for knowledge base data access, verify permissions directly at the data source rather than relying on intermediate systems.
  - When using the search engine example, access verification occurs when retrieving the actual result from the data source, not during the initial search.
  - For vector databases, the generative AI application validates access rights by sending an authorization request to the data source before retrieving the data.
  - This helps make sure that the data source that maintains the authoritative access control rules determines whether the principal has permission to access specific objects.
  - This real-time authorization check means permission changes are immediately reflected when accessing the data source.
  - This authorization pattern is similar to how AWS Lake Formation manages access to structured data.
  - Lake Formation evaluates permissions when a principal requests access to databases or tables, granting or denying access based on the principal's defined permissions.
  - You can implement comparable authorization controls for vector database results before providing that context to large language models.
* **Technical Entities (Classes/Functions/APIs):** `AWS Lake Formation`

## Solution overview: S3 Access Grants with Bedrock Knowledge Bases
* **Key Points:**
  - In the following example, you have an ACME organization that wants to create a generative AI chatbot for their employees.
  - There are multiple teams within the organization (Marketing, Sales, HR, and IT) that work on projects throughout the organization.
  - Each principal will have access to their respective project (for example /projects/projectA) or department folders (for example departments/marketing/).
  - Marketing also will have access to everything in the projects folder (/projects/*) unless they are considered highly confidential files.
  - To mark Project B files as highly confidential, you will include a metadata tag for objects within the Project C prefix with classification = 'highly confidential'.
  - To authorize access for each principal to the objects within the knowledge base, you will use Amazon S3 Access Grants.
  - You can learn how to set up S3 Access Grants in Part 1 or Part 2 of the blog series.
  - Within AWS IAM Identity Center, you will add each user to their respective groups.
* **Technical Entities (Classes/Functions/APIs):** `Amazon S3 Access Grants`, `IAM Identity Center`
* **Key Points:**
  - The application flow is the following, as shown in Figure 4:
    - The user uses their identity provider (IdP) to sign in to the generative AI application (steps 1a, 1b, and 1c).
    - The generative AI application exchanges a token with IAM Identity Center and assumes the role on behalf of the user (step 2).
    - The generative AI application calls S3 Access Grants to get a list of the grants the user is authorized to access (step 3).
    - The user sends a query to the generative AI application (step 4).
    - The generative AI application sends a query to knowledge base (step 5).
    - The generative AI application reviews chunks from the knowledge base against the scopes the user is authorized to access (step 6).
    - Only scopes the user is authorized to will be passed to the LLM for a response (step 7).
    - The generative AI application will continue steps 5–7 until you want to get a new list of authorized scopes (repeat step 4) or the token expires (repeat steps 3 and 4).
* **Technical Entities (Classes/Functions/APIs):** `RetrievalFilter`, `retrievalConfiguration`
* **Code Snippet:**
```json
{
    "metadataAttributes" : { 
        "classification" : "highly confidential"
    }
}
```
* **Key Points:**
  - When you call the knowledge base without performing any data authorization, you receive the following back when asking "What is the status of my project."
  - With each object within the data source, you also include meta data, in the form of *.metadata.json, which is used by the knowledge base to assign specific key/value pairs to each object.
  - You pass this filter as part of the Bedrock knowledge base request, using a RetrievalFilter within the retrievalConfiguration.
* **Technical Entities (Classes/Functions/APIs):** `assume_role()`, `get_caller_grant_scopes()`, `check_grant_scopes()`
* **Code Snippet:**
```python
# Execute the workflow
# 1. Assume role for S3 access
client_s3_oidc = assume_role(
   args.client_id, args.grant_type, args.assertion,
   args.role_arn, args.role_session_name, args.provider_arn
)
    
# 2. Get caller's authorized S3 scopes
scopes = get_caller_grant_scopes(client_s3_oidc, args.account)
        
# 3. Filter chunks based on caller's authorization
authorized, not_authorized = check_grant_scopes(chunks, scopes)
```

### Step 1: User uses the IdP to sign in to the generative AI application
* **Key Points:**
  - When Bob first accesses the generative AI application, the application will redirect him using a single sign-on flow for him to authenticate with their IdP.
  - Bob will receive a signed identity token from the IdP that will validate who Bob is from an identity perspective.
* **Technical Entities (Classes/Functions/APIs):** `IdP`, `identity token`
* **Code Snippet:**
```json
{
    "sub": "sub",
    "email": "bob@example.com",
    "aud": "bob",
    "iss": "https://tokens.identity-solutions.example.com",
    "exp": 1744219319,
    "iat": 1744218719,
    "name": "bob"
}
```

### Step 2: Token exchange with IAM Identity Center
* **Key Points:**
  - After Bob is authenticated and passes his token to the generative AI application, the application will exchange the identity token from the IdP with the IAM Identity Center identity token and retrieve temporary credentials on behalf of Bob.
  - You will create a function called assume_role in Python that passes multiple different variables used to allow Bob to assume a role inside AWS
* **Technical Entities (Classes/Functions/APIs):** `IAM Identity Center`, `assume_role()`, `create_token_with_iam()`, `assume_role()`, `sso-oidc`, `sts`, `s3-control`
* **Code Snippet:**
```python
def assume_role(client_id, grant_type, client_assertion, role_arn, role_session_name, provider_arn):
    """
    Assume an IAM role using SSO/OIDC authentication and return an S3 control client.
    
    Args:
        client_id: The ID of the OIDC client
        grant_type: The type of grant being requested
        client_assertion: The client assertion token
        role_arn: ARN of the role to assume
        role_session_name: Name for the temporary session
        provider_arn: ARN of the identity provider
        
    Returns:
        boto3.client: An S3 control client with temporary credentials
    """
    client_oidc = boto3.client('sso-oidc')
    client_sts = boto3.client('sts')
    try:
        # Get ID token from IAM using SSO OIDC
        token_response = client_oidc.create_token_with_iam(
            clientId=client_id,
            grantType=grant_type,
            assertion=client_assertion
        )
        
        # Extract identity context from token
        id_token = jwt.decode(token_response['idToken'], options={'verify_signature': False})
        identity_context = id_token['sts:identity_context']
        
        # Assume role using identity context
        temp_credentials = client_sts.assume_role(
            RoleArn=role_arn,
            RoleSessionName=role_session_name,
            ProvidedContexts=[{
                'ProviderArn': provider_arn,
                'ContextAssertion': identity_context
            }]
        )
        
        # Create and return S3 control client with temporary credentials
        creds = temp_credentials['Credentials']
        return boto3.client(
            's3control',
            region_name='us-west-2',
            aws_access_key_id=creds['AccessKeyId'],
            aws_secret_access_key=creds['SecretAccessKey'],
            aws_session_token=creds['SessionToken']
        )
    except ClientError as e:
        print(f'Error: {e}')
        sys.exit(1)
```

### Step 3: Retrieve the caller grant scopes
* **Key Points:**
  - Next, you need to retrieve what Bob is allowed to access in the data source by using S3 Access Grants.
  - In our example, you need to validate the data Bob is authorized to access with the data source, not the S3 object itself.
  - To obtain the prefixes Bob is authorized to access, you will need to do the following in the get_caller_grant_scopes function.
  - First, you will pass the s3control client that was returned from assume_role. in addition to the account for the S3 access grants.
  - With the temporary role for Bob, you will call list_caller_access_grants.
  - This will return a list of caller access grants available to Bob.
  - You add the scopes to an array and return the array back to the application.
* **Technical Entities (Classes/Functions/APIs):** `S3 Access Grants`, `get_caller_grant_scopes()`, `list_caller_access_grants()`, `s3control`
* **Code Snippet:**
```json
{
    "ResponseMetadata": {
        ...
    },
    "CallerAccessGrantsList": [
        {
            "Permission": "READ",
            "GrantScope": "s3:// amzn-s3-demo-bucket/departments/sales/*",
            "ApplicationArn": "ALL"
        },
        {
            "Permission": "READ",
            "GrantScope": "s3:// amzn-s3-demo-bucket/projects/projectA/*",
            "ApplicationArn": "ALL"
        }
    ]
}
```
```python
def get_caller_grant_scopes(client, account):
    """
    Retrieve the S3 access scopes granted to a caller.
    
    Args:
        client: S3 control client with assumed role credentials
        account: AWS account ID
        
    Returns:
        List of S3 path prefixes the caller is authorized to access
    """
    try:
        # Get list of access grants for the caller
        response = client.list_caller_access_grants(AccountId=account)
        
        # Extract S3 path prefixes and remove trailing wildcards
        scopes = [grant['GrantScope'].replace('*','') for grant in response['CallerAccessGrantsList']]
        return scopes
    except ClientError as e:
        print(f'Error: {e}')
        sys.exit(1)
```

### Step 4: Check caller grant scopes
* **Key Points:**
  - The last step is to check chunks returned by the knowledge base against the list of the grants Bob has access to.
  - For this, you define check_grant_scopes and pass both the chunks and the scopes Bob is authorized to access.
  - The variable chunks is an array of dictionaries that you will parse, validating it against the list of scopes, shown in the following code example.
  - You first loop through each chunk that was passed to the function.
  - For each chunk, you will check to see if the chunk location starts with a given prefix that is in the S3 access grant.
  - If a match is found, you add it to the chunk, along with the scope found in the S3 access grant, to the list of e chunks.
  - If a match is not found in the scopes, then you add it to the not_authorized chunks.
  - The function will return both the list of authorized chunks and not_authorized chunks to provide visibility into the different chunks Bob was denied access to.
* **Technical Entities (Classes/Functions/APIs):** `check_grant_scopes()`
* **Code Snippet:**
```python
def check_grant_scopes(chunks, scopes):
    """
    Check which chunks a user is authorized to access based on their granted scopes.
    
    Args:
        chunks: List of dictionaries containing content chunks with 'location' keys
        scopes: List of authorized S3 path prefixes the user has access to
        
    Returns:
        tuple: (authorized_chunks, unauthorized_chunks)
    """
    authorized = []
    not_authorized = []
    # If user has no scopes, they are not authorized for any chunks
    if not scopes:
        return [], chunks
    
    # Check each chunk against available scopes
    for chunk in chunks:
        location = chunk['location']
        authorized_scope = next((scope for scope in scopes if location.startswith(scope)), None)
        
        if authorized_scope:
            chunk['scope'] = authorized_scope
            authorized.append(chunk)
        else:
            not_authorized.append(chunk)
    
    return authorized, not_authorized
```

## Solution considerations
* **Key Points:**
  - When implementing this authorization architecture for RAG implementations, it's important to understand several key considerations that impact security, performance, and scalability.
  - These considerations help make sure your implementation maintains strong security controls, while optimizing system performance and providing flexibility for different data sources.
  - The following points outline important aspects to evaluate when designing and implementing this authorization pattern:
    - For this example, you used S3 Access Grants as the example of how to check for authorization. However, this architecture can be used with your choice of data source, if the URI for the data source is returned from the knowledge base and there is an API that can be called to validate what a principal is authorized to access, like the get_caller_grant_scopes function described previously.
    - The use of S3 Access Grants provides authorization for a principal to access the data source. Additional access control policies could be applied to each bucket by adding a key/value tag or data source if desired. By doing this, the principal would be denied access to the bucket even though S3 Access Grants provides authorization. To support this functionality, you can add metadata for the vector database to ingest and filter on the query to the knowledge base, as shown in the preceding example.
    - Similar to stale data until resync of the knowledge base, the list of authorized scopes can also become stale. It's up to you to decide how often you refresh the list of authorized scopes (step 3 in Figure 4) and the duration of the assume role of the principal (step 2 in Figure 4).
    - Depending on the chunks the principal is authorized to access and what the knowledge base returns, chunks could be dropped before sending to the LLM. From a security point of view, this is preferred so principals will not get access to chunks they aren't authorized to. From an architecture point of view, you should optimize the knowledge base query and add additional metadata tags to limit the number of non-authorized chunks returned from the knowledge base. This is one reason to include a not_authorized list as part of the check_grant_scopes function.
* **Technical Entities (Classes/Functions/APIs):** `get_caller_grant_scopes()`, `check_grant_scopes()`

## Conclusion
* **Key Points:**
  - In this post, I showed you an architecture pattern to provide strong authorization for results returned from knowledge bases.
  - You walked through the importance of strong authorization with knowledge bases and how to implement authorization with Amazon S3 Access Grants.
  - Lastly, you walked through code examples of how this would work in practice using Amazon Bedrock Knowledge Bases with S3 Access Grants.
  - For additional information on generative AI security, take a look at other posts in the AWS Security Blog and AWS blog posts covering generative AI.
* **Technical Entities (Classes/Functions/APIs):** `Amazon S3 Access Grants`, `Amazon Bedrock Knowledge Bases`
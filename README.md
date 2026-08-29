# Data-query integration

Infrastructure-as-code for a small "data query" integration on AWS:

- an **HTTP API** and a **Lambda** tool that returns records from a database via the RDS Data API,
- a **Bearer-token secret** the Lambda validates,
- the **IAM** for the Lambda and an execution role for a Bedrock AgentCore assistant that calls the Lambda as a tool.

Everything is defined in a single AWS SAM template: [`template.yaml`](./template.yaml).
The database itself is **not** created by the template — pass an existing data source as parameters.

## Deploy

With the SAM CLI:

```bash
sam deploy --guided
```

Or with plain CloudFormation (no SAM CLI — the Serverless transform runs server-side):

```bash
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name data-query \
  --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
  --parameter-overrides ApiToken=<token> DbClusterArn=<arn> DbSecretArn=<arn> DbName=companydb
```

The stack Outputs print the public endpoint. Test it:

```bash
curl -X POST "<ApiUrl>" -H "Authorization: Bearer <token>"
```

## Notes

- Every resource name is a parameter (defaults are the real names), so the same template can deploy a
  parallel copy in the same account/region by overriding the `*Name` parameters.
- Everything the stack creates is tracked by CloudFormation and tagged with the `Project` parameter.
- The Bedrock AgentCore agent runtime runs a container image that must be built and pushed to ECR
  separately (see the commented section in `template.yaml`); the template provisions its execution role.

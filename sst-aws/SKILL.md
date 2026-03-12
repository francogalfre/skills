---
name: sst-aws
description: Expert guidance for building and deploying full-stack applications on AWS using SST v3 (Ion). Use when working with `sst.config.ts`; deploying Lambda functions, APIs, queues, buckets, tables, or databases; linking resources; managing secrets; running `sst dev`; handling multi-stage deployments; hardening security/cost defaults; troubleshooting deploy/dev issues; or migrating from SST v2 to v3.
---

# SST v3 (Ion) — AWS Full-Stack Deployment

SST (Serverless Stack) v3, codenamed Ion, is a TypeScript-first Infrastructure
as Code (IaC) framework for deploying full-stack applications to AWS and other
cloud providers. It uses Pulumi and Terraform under the hood instead of AWS CDK
(which was used in v2), enabling faster deployments and multi-cloud support.

> **Note:** SST v3 is production-viable. Treat the docs as the source of truth
> for exact APIs, and keep your skill usage focused on patterns that stay stable:
> stages, linking, secrets, least-privilege permissions, and safe removal policies.

---

## When to Apply This Skill

- Writing or modifying `sst.config.ts`
- Deploying Lambda functions, APIs, or frontends to AWS
- Working with SST components: `Function`, `Api`, `Bucket`, `Queue`, `Cron`, `Postgres`, `NextjsSite`, `VPC`, etc.
- Linking AWS resources together (e.g. connecting a Lambda to an S3 bucket)
- Managing secrets and environment variables in SST
- Setting up multi-stage environments (dev, staging, production)
- Using `sst dev` for live local development
- Migrating from SST v2 to v3

---

## Workflow: Build a production-ready SST app (AWS)

Follow this flow when creating or updating an SST project so you don’t miss the sharp edges:

1. **Confirm basics**: AWS account, region, credentials/profile, and stage naming strategy.
2. **Define app defaults** in `app()` (name, stage, removal policy, home/provider).
3. **Model your resources** in `run()` using SST components (Api, Function, Bucket, Queue, Dynamo, Postgres, etc.).
4. **Link resources** instead of hardcoding ARNs/URLs/connection strings; access them via `Resource.*` in code.
5. **Secrets**: use `sst.Secret` + `sst secret set` (never `environment: {}` for sensitive values).
6. **Dev loop**: prefer `sst dev` during development; validate that the stage is isolated and safe to remove.
7. **Prod safety**: set `removal: "retain"` for production; explicitly size/scaling/timeouts for prod workloads.
8. **Troubleshoot** using the checklist in [Troubleshooting](#troubleshooting).
9. **If migrating v2 → v3**: map constructs + refactor env vars to linking (`Resource.*`).

---

## Details to collect before you write `sst.config.ts`

If the user didn’t provide these, ask (or infer from repo context):

| Detail               | Why it matters                         | Example                        |
| -------------------- | -------------------------------------- | ------------------------------ |
| **App name**         | Namespaces resources and outputs       | `my-app`                       |
| **Stages**           | Isolation + deletion safety            | `dev`, `staging`, `production` |
| **AWS region**       | Latency, pricing, service availability | `us-east-1`                    |
| **AWS auth**         | Which credentials/profile to use       | `AWS_PROFILE=personal`         |
| **Workload shape**   | Timeouts/memory/concurrency            | API + background jobs          |
| **Data stores**      | RDS vs Dynamo, consistency, scaling    | Postgres + Dynamo              |
| **Networking needs** | VPC vs public, DB access               | RDS in VPC                     |
| **Frontend**         | Next.js/static, env vars               | `Nextjs` with `NEXT_PUBLIC_*`  |
| **Compliance**       | Encryption, retention, audit           | KMS, log retention             |

---

## Core Concepts

### sst.config.ts — The Entry Point

Every SST app is configured in `sst.config.ts` at the root of the project.

```ts
///

export default $config({
  app(input) {
    return {
      name: "my-app",
      removal: input?.stage === "production" ? "retain" : "remove",
      home: "aws",
    };
  },
  async run() {
    // Define all resources here
    const bucket = new sst.aws.Bucket("MyBucket");

    const api = new sst.aws.ApiGatewayV2("MyApi");
    api.route("GET /", {
      handler: "src/api/hello.handler",
      link: [bucket],
    });
  },
});
```

### Stages

SST uses stages to separate environments. Each developer gets their own
isolated stage in dev, and you deploy to named stages like `staging` or
`production`.

```bash
# Deploy to your personal dev stage
sst deploy

# Deploy to staging
sst deploy --stage staging

# Deploy to production
sst deploy --stage production
```

Access the current stage inside `sst.config.ts`:

```ts
app(input) {
  return {
    name: "my-app",
    // Retain resources in production to avoid accidental deletion
    removal: input?.stage === "production" ? "retain" : "remove",
    home: "aws",
  };
}
```

---

## Production safety defaults (high impact)

Use these defaults unless you have a reason not to:

- **Removal policy**: retain production stateful resources.
- **Least privilege**: grant permissions by linking resources (avoid wildcard IAM).
- **Separate state per stage**: never share prod and dev databases/buckets.
- **Explicit timeouts**: set API, queue visibility, and Lambda timeouts intentionally.
- **Cost controls**: scale-to-zero only where safe (e.g., dev DB), be careful with prod cold starts.

Example (retain in prod):

```ts
export default $config({
  app(input) {
    const stage = input?.stage;
    const isProd = stage === "production";
    return {
      name: "my-app",
      removal: isProd ? "retain" : "remove",
      home: "aws",
    };
  },
  async run() {
    // ...
  },
});
```

---

## Common Components

### Lambda Function

```ts
const fn = new sst.aws.Function("MyFunction", {
  handler: "src/functions/hello.handler",
  runtime: "nodejs20.x",
  timeout: "30 seconds",
  memory: "512 MB",
  environment: {
    STAGE: $app.stage,
  },
});
```

### API Gateway v2 (HTTP API)

```ts
const api = new sst.aws.ApiGatewayV2("MyApi", {
  cors: true, // enable CORS for frontend consumption
});

api.route("GET /users", "src/api/users/list.handler");
api.route("POST /users", "src/api/users/create.handler");
api.route("GET /users/{id}", "src/api/users/get.handler");

// Export the URL as a stack output
return {
  apiUrl: api.url,
};
```

### S3 Bucket

```ts
const bucket = new sst.aws.Bucket("AssetsBucket", {
  public: false, // set true only for public static files
});
```

### SQS Queue

```ts
const queue = new sst.aws.Queue("JobQueue");

// Subscribe a Lambda to process messages
queue.subscribe("src/workers/processJob.handler", {
  batch: {
    size: 10,
    window: "20 seconds",
  },
});
```

### Cron Job

```ts
new sst.aws.Cron("DailyReport", {
  job: "src/cron/dailyReport.handler",
  schedule: "cron(0 8 * * ? *)", // every day at 8am UTC
});
```

### RDS Postgres

```ts
const db = new sst.aws.Postgres("MyDatabase", {
  scaling: {
    min: "0 ACU", // scale to zero when idle (saves cost)
    max: "4 ACU",
  },
});
```

### DynamoDB Table

```ts
const table = new sst.aws.Dynamo("UsersTable", {
  fields: {
    pk: "string",
    sk: "string",
  },
  primaryIndex: { hashKey: "pk", rangeKey: "sk" },
});
```

### Next.js Site

```ts
const site = new sst.aws.Nextjs("MyNextApp", {
  path: "packages/web", // path to Next.js app in monorepo
  environment: {
    NEXT_PUBLIC_API_URL: api.url,
  },
});

return {
  siteUrl: site.url,
};
```

---

## Resource Linking

Resource linking is one of SST's most powerful features. It lets you connect
resources together without hardcoding ARNs, names, or connection strings.

```ts
const bucket = new sst.aws.Bucket("UploadsBucket");
const table = new sst.aws.Dynamo("ItemsTable", { ... });

const api = new sst.aws.ApiGatewayV2("Api");

api.route("POST /upload", {
  handler: "src/api/upload.handler",
  link: [bucket, table], // grants permissions + injects config automatically
});
```

Access linked resources in your Lambda using the SST SDK:

```ts
// src/api/upload.handler.ts
import { Resource } from "sst";
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";

const s3 = new S3Client({});

export const handler = async (event: any) => {
  await s3.send(
    new PutObjectCommand({
      Bucket: Resource.UploadsBucket.name, // type-safe, no hardcoding
      Key: "file.txt",
      Body: "Hello world",
    }),
  );

  return { statusCode: 200 };
};
```

---

## Secrets Management

Never hardcode secrets. Use SST's built-in secrets system.

```bash
# Set a secret (stored encrypted in AWS SSM)
sst secret set DatabasePassword "my-secret-password"
sst secret set DatabasePassword "prod-password" --stage production
```

```ts
// sst.config.ts
const dbPassword = new sst.Secret("DatabasePassword");

const fn = new sst.aws.Function("MyFunction", {
  handler: "src/handler.main",
  link: [dbPassword],
});
```

```ts
// src/handler.ts
import { Resource } from "sst";

export const main = async () => {
  const password = Resource.DatabasePassword.value; // type-safe
};
```

---

## Live Development (`sst dev`)

`sst dev` deploys real AWS resources and proxies Lambda invocations to your
local machine, giving you live debugging with actual cloud events.

```bash
sst dev
```

In a monorepo, `sst dev` runs a multiplexer that starts all your dev servers
automatically — you don't need to run your frontend separately.

### Tips for `sst dev`

- Your Lambda code runs locally but the trigger (API Gateway, SQS, etc.) is real AWS
- Use `console.log` freely — logs show directly in your terminal
- Changes to Lambda code take effect immediately without redeployment
- You still need a deployed stage — `sst dev` deploys infrastructure on first run

---

## CI/CD notes (minimal, reliable)

- Prefer **explicit stages** in CI: `sst deploy --stage staging` and `sst deploy --stage production`.
- Use a **dedicated AWS role/user** for CI with least privilege.
- Treat `sst.config.ts` changes like infra changes: review diffs carefully and deploy to staging first.

---

## Multi-Stage Best Practices

```ts
async run() {
  const isProd = $app.stage === "production";

  const db = new sst.aws.Postgres("Database", {
    scaling: {
      min: isProd ? "2 ACU" : "0 ACU", // scale to zero in dev to save cost
      max: isProd ? "16 ACU" : "4 ACU",
    },
  });

  const queue = new sst.aws.Queue("JobQueue", {
    // longer visibility timeout in production
    visibilityTimeout: isProd ? "5 minutes" : "30 seconds",
  });
}
```

---

## Project Structure

Recommended structure for a full-stack SST app or monorepo:

```
my-app/
├── sst.config.ts           # SST entry point — all infra defined here
├── package.json
├── packages/
│   ├── functions/          # Lambda handlers
│   │   └── src/
│   │       ├── api/
│   │       │   └── users.ts
│   │       └── workers/
│   │           └── processJob.ts
│   ├── web/                # Next.js or other frontend
│   │   └── ...
│   └── core/               # Shared business logic (no AWS deps)
│       └── src/
│           └── domain/
└── .sst/                   # Auto-generated by SST — do not edit
```

Keep Lambda handlers thin — put business logic in `packages/core` so it's
testable without AWS.

---

## Common Patterns

### Pattern 1 — API + Database + Auth

```ts
async run() {
  const db = new sst.aws.Postgres("Database");

  const api = new sst.aws.ApiGatewayV2("Api", { cors: true });

  api.route("GET /health", "packages/functions/src/health.handler");

  api.route("GET /users", {
    handler: "packages/functions/src/api/users.list",
    link: [db],
  });

  const site = new sst.aws.Nextjs("Web", {
    environment: { NEXT_PUBLIC_API_URL: api.url },
  });

  return { apiUrl: api.url, siteUrl: site.url };
}
```

### Pattern 2 — Background Job with Queue

```ts
async run() {
  const queue = new sst.aws.Queue("EmailQueue");

  // Producer: API route that enqueues jobs
  const api = new sst.aws.ApiGatewayV2("Api");
  api.route("POST /send-email", {
    handler: "src/api/sendEmail.handler",
    link: [queue],
  });

  // Consumer: worker that processes the queue
  queue.subscribe({
    handler: "src/workers/emailWorker.handler",
    timeout: "2 minutes",
  });
}
```

### Pattern 3 — S3 Event Trigger

```ts
async run() {
  const bucket = new sst.aws.Bucket("UploadsBucket");

  // Trigger a Lambda when a file is uploaded
  bucket.subscribe("src/workers/processUpload.handler", {
    events: ["s3:ObjectCreated:*"],
    filterPrefix: "uploads/",
  });
}
```

---

## Migrating from SST v2 to v3

Key differences to be aware of:

| SST v2                            | SST v3                                 |
| --------------------------------- | -------------------------------------- |
| Based on AWS CDK + CloudFormation | Based on Pulumi + Terraform            |
| `new Api(...)`                    | `new sst.aws.ApiGatewayV2(...)`        |
| `new Table(...)`                  | `new sst.aws.Dynamo(...)`              |
| `new Bucket(...)`                 | `new sst.aws.Bucket(...)`              |
| `new NextjsSite(...)`             | `new sst.aws.Nextjs(...)`              |
| `sst.config.ts` with `stacks()`   | `sst.config.ts` with `run()`           |
| `process.env.BUCKET_NAME`         | `Resource.BucketName.name` via SST SDK |
| Slow CloudFormation deploys       | Fast Pulumi/Terraform deploys          |

- Importing existing v2 resources into v3 is supported but requires manual work
- If heavily reliant on CDK L3 constructs, some may not have Pulumi equivalents yet
- Check the official SST v3 migration guide: https://sst.dev/docs/migrate/v2

---

## Common Mistakes to Avoid

| Mistake                                       | Problem                           | Fix                                                      |
| --------------------------------------------- | --------------------------------- | -------------------------------------------------------- |
| Hardcoding ARNs or resource names             | Breaks across stages              | Use `Resource.X.name` via SST SDK after linking          |
| Putting secrets in `environment: {}`          | Visible in Lambda console         | Use `sst.Secret` instead                                 |
| Not setting `removal: "retain"` in production | Resources deleted on `sst remove` | Always set `retain` for production databases and buckets |
| One giant `sst.config.ts`                     | Hard to maintain                  | Split into files and import into `run()`                 |
| Not using `sst dev`                           | Slow feedback loop                | Always use live dev mode during development              |
| Missing `cors: true` on API                   | Frontend can't call API           | Set CORS on `ApiGatewayV2` or configure manually         |
| Scaling RDS to minimum in production          | Cold starts on idle DB            | Set `min: "2 ACU"` or higher in production               |

---

## Troubleshooting

Quick checks that solve most SST+AWS issues:

- **`sst dev` or deploy fails with auth errors**: confirm `AWS_PROFILE` / env credentials and the target account; avoid mixing profiles across terminals.
- **Resources not found at runtime**: ensure you used `link: [...]` and are reading via `Resource.*` (not hardcoded names/ARNs).
- **Secrets are `undefined`**: confirm `sst secret set ...` was run for the same `--stage` you’re deploying.
- **CORS issues**: enable CORS on `ApiGatewayV2` or configure allowed origins/headers explicitly.
- **Queue consumer not firing**: verify subscription exists, visibility timeout vs function timeout, and check DLQ/CloudWatch logs.
- **RDS cold starts**: don’t use scale-to-zero in production unless you accept latency; increase `min` ACU.

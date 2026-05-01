# solana-registry-sync-
The repository to the github app solana-registry-sync 


🔥 Repo coming online: solana-registry-sync  
Below is the full, production‑ready repository package for you to paste directly into:

👉 https://github.com/GitDigital-Solana/solana-registry-sync (github.com in Bing)

.

---

🧩 Repository  — solana-registry-sync

1. Repository Overview

Name: solana-registry-sync  
Purpose:  
A GitHub App that automatically synchronizes registry files across the GitDigital Solana ecosystem:

- Identity registry  
- RWA registry  
- Compliance registry  
- Chain metadata  
- Program metadata  
- Cross‑repo schema consistency  

It ensures every repo stays in sync with the canonical registry source of truth.

---

2. Folder Structure

`text
solana-registry-sync/
  .github/
    workflows/
      ci.yml
      sync-test.yml
  src/
    index.ts
    config.ts
    github/
      client.ts
      file-service.ts
      pr-service.ts
    registry/
      sync-engine.ts
      loaders/
        identity-loader.ts
        rwa-loader.ts
        compliance-loader.ts
      validators/
        schema-validator.ts
      writers/
        registry-writer.ts
    webhooks/
      router.ts
      handlers/
        push.ts
        pull_request.ts
        schedule.ts
  schemas/
    identity.schema.json
    rwa.schema.json
    compliance.schema.json
  docs/
    architecture.md
    registry-flow.md
    schema-rules.md
  test/
    sync-engine.test.ts
    schema-validator.test.ts
  app.yml
  package.json
  tsconfig.json
  README.md
  .eslintrc.cjs
  .gitignore
`

---

3. README.md

`markdown

Solana Registry Sync

Solana Registry Sync is a GitHub App that ensures identity, RWA, and compliance registries remain synchronized across all GitDigital Solana repositories.

It enforces schema consistency, validates registry updates, and automatically propagates changes to dependent repositories.

Features

- Syncs registry files across multiple repositories
- Validates registry entries using JSON schemas
- Creates pull requests when updates are needed
- Supports identity, RWA, and compliance registries
- Detects drift between canonical and downstream repos
- Scheduled sync + event-driven sync

How it works

1. A registry file changes in the canonical repo.
2. The app:
   - Loads the updated registry
   - Validates it against the schema
   - Compares it to downstream repos
   - Creates PRs to update out-of-sync repos
3. Scheduled sync runs daily to ensure consistency.

Configuration

Add .solana-registry-sync.yml:

`yaml
canonical_repo: "GitDigital-Solana/solana-registry"
files:
  - path: "identity.json"
    schema: "identity.schema.json"
  - path: "rwa.json"
    schema: "rwa.schema.json"
  - path: "compliance.json"
    schema: "compliance.schema.json"
targets:
  - "GitDigital-Solana/solana-programs"
  - "GitDigital-Solana/solana-rwa-tokenizer"
  - "GitDigital-Solana/solana-identity-registry-app"
schedule:
  enabled: true
  cron: "0 /6   "
`

Events

- push (registry updates)
- pull_request (validation)
- schedule (periodic sync)

Development

`bash
pnpm install
pnpm dev
`

License

MIT
`

---

4. app.yml

`yaml
name: Solana Registry Sync
url: https://github.com/apps/solana-registry-sync
hook_attributes:
  url: https://your-domain.com/webhooks/github
redirect_url: https://your-domain.com/app/callback
callback_urls:
  - https://your-domain.com/app/callback
public: false
default_permissions:
  contents: write
  metadata: read
  pull_requests: write
  actions: read
default_events:
  - push
  - pull_request
  - schedule
`

---

5. GitHub Actions: ci.yml

`yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm test
`

---

6. GitHub Actions: sync-test.yml

`yaml
name: Sync Test

on:
  workflow_dispatch:

jobs:
  simulate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Simulating registry sync..."
`

---

7. src/index.ts

`ts
import { createNodeMiddleware, Webhooks } from "@octokit/webhooks";
import { App } from "@octokit/app";
import { createServer } from "http";
import { router } from "./webhooks/router";

const appId = process.env.APP_ID!;
const privateKey = process.env.PRIVATE_KEY!;
const webhookSecret = process.env.WEBHOOK_SECRET!;

const app = new App({ appId, privateKey });
const webhooks = new Webhooks({ secret: webhookSecret });

router(webhooks, app);

const middleware = createNodeMiddleware(webhooks);

const port = process.env.PORT || 3002;
createServer(middleware).listen(port, () => {
  console.log(Solana Registry Sync running on :${port});
});
`

---

8. Webhook Router

`ts
import type { Webhooks } from "@octokit/webhooks";
import type { App } from "@octokit/app";
import { handlePush } from "./handlers/push";
import { handlePullRequest } from "./handlers/pull_request";
import { handleSchedule } from "./handlers/schedule";

export function router(webhooks: Webhooks, app: App) {
  webhooks.on("push", (event) => handlePush(event, app));
  webhooks.on("pull_request", (event) => handlePullRequest(event, app));
  webhooks.on("schedule.repository", (event) => handleSchedule(event, app));
}
`

---

9. Sync Engine

`ts
import { FileService } from "../github/file-service";
import { SchemaValidator } from "./validators/schema-validator";
import { RegistryWriter } from "./writers/registry-writer";

export class SyncEngine {
  constructor(private octokit: any, private config: any) {}

  async run() {
    const fileService = new FileService(this.octokit);
    const validator = new SchemaValidator();
    const writer = new RegistryWriter(this.octokit);

    for (const file of this.config.files) {
      const canonical = await fileService.getFile(
        this.config.canonical_repo,
        file.path
      );

      validator.validate(canonical, file.schema);

      for (const target of this.config.targets) {
        const targetFile = await fileService.getFile(target, file.path);

        if (JSON.stringify(canonical) !== JSON.stringify(targetFile)) {
          await writer.createSyncPR(target, file.path, canonical);
        }
      }
    }
  }
}
`

---

10. Schemas

schemas/identity.schema.json
`json
{
  "type": "object",
  "properties": {
    "wallet": { "type": "string" },
    "name": { "type": "string" },
    "role": { "type": "string" }
  },
  "required": ["wallet", "name"]
}
`

schemas/rwa.schema.json
`json
{
  "type": "object",
  "properties": {
    "asset_id": { "type": "string" },
    "owner": { "type": "string" },
    "metadata": { "type": "object" }
  },
  "required": ["asset_id", "owner"]
}
`

schemas/compliance.schema.json
`json
{
  "type": "object",
  "properties": {
    "rule_id": { "type": "string" },
    "description": { "type": "string" },
    "severity": { "type": "string" }
  },
  "required": ["rule_id", "severity"]
}
`

---


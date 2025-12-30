# Text-to-Video Service Architecture

Followed with **how you actually enforce this**, because patterns without enforcement rot fast.

## Goal

Provide a stable, testable, and provider-agnostic way to generate videos from text prompts.

This system must:
- Avoid direct SDK usage in UI code
- Isolate vendor churn
- Support multiple providers
- Degrade gracefully when providers fail
- Produce auditability and health signals

---

## High-Level Pattern

This architecture is split into three planes:

1. Runtime Adapters (provider-specific)
2. Orchestration Service (provider-agnostic)
3. Control Plane (visibility and governance)

Only the orchestration service is exposed to application code.

---

## 1. Provider Adapters (Runtime Plane)

Each external provider is wrapped by its own adapter.

Adapters:
- Import exactly one SDK
- Implement a shared internal interface
- Handle retries, timeouts, auth, and error normalization
- Emit lifecycle events

Adapters do NOT:
- Call other adapters
- Contain routing logic
- Expose SDK-specific types

### Adapter Interface

```ts
export interface TextToVideoProvider {
  generateVideo(input: VideoPrompt): Promise<VideoResult>;
  healthCheck(): Promise<HealthStatus>;
  capabilities(): ProviderCapabilities;
}
````

### Example Adapters

* `GoogleVeoAdapter`
* `RunwayAdapter`
* `PikaAdapter`

Adapters live in `/providers/*`.

---

## 2. Orchestration Service (Runtime Plane)

The orchestration service coordinates requests.

Responsibilities:

* Select provider
* Enforce policy (quotas, allowlists, flags)
* Emit domain events
* Return normalized results

The orchestration layer:

* Never imports provider SDKs
* Never exposes provider identity unless explicitly requested
* Treats providers as interchangeable

### Example Flow

1. Client calls `POST /video`
2. Orchestrator emits `VideoRequested`
3. Provider is selected
4. Adapter executes request
5. Orchestrator emits `VideoGenerated` or `VideoFailed`
6. Normalized result returned

---

## 3. Service Registry and Dashboard (Control Plane)

This plane is informational only.

It tracks:

* Provider name
* SDK version
* Repo or CDN link
* Capabilities
* Health check results
* Error rates
* Last successful job

This plane:

* Is never queried during runtime requests
* Is populated by scheduled jobs and event consumers
* Exists to inform humans, not code

---

## Non-Goals

* No shared "AI SDK service"
* No frontend imports of provider SDKs
* No synchronous provider chaining
* No provider-specific types leaking upward

---

## Failure Model

* Provider failure must not crash the orchestrator
* Partial outages are acceptable
* Timeouts are treated as first-class failures
* All failures emit events

---

## Summary

Loose coupling is enforced through:

* Adapter isolation
* Event-based signaling
* Contract-first interfaces
* Clear separation of runtime and control planes

````

---

## Now the important part: **how you enforce this**

Patterns don’t survive by documentation.  
They survive by **friction and tests**.

### 1. Dependency enforcement (this is non-negotiable)

**Rule:**  
Front-end and orchestration code must not import provider SDKs.

**How to enforce:**

**ESLint `no-restricted-imports`** - catches violations at PR time:

```js
// .eslintrc.js
no-restricted-imports: [
  "error",
  {
    "paths": [
      {
        "name": "@google/genai",
        "message": "Import SDKs only inside provider adapters (/providers/*)"
      },
      {
        "name": "@runwayml/sdk",
        "message": "Import SDKs only inside provider adapters (/providers/*)"
      },
      {
        "name": "@pika/sdk",
        "message": "Import SDKs only inside provider adapters (/providers/*)"
      },
      {
        "pattern": "**/providers/**",
        "group": ["**/orchestrator/**", "**/ui/**"],
        "message": "Orchestrator and UI cannot import provider adapters directly"
      }
    ]
  }
]
```

**TypeScript path mapping** - enforce at compile time:

```json
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@providers/*": ["./src/providers/*"],
      "@orchestrator/*": ["./src/orchestrator/*"]
    }
  }
}
```

**CI/CD check** - fail builds that violate boundaries:

```bash
# scripts/check-boundaries.sh
#!/bin/bash
# Check for SDK imports outside /providers
if grep -r "@google/genai\|@runwayml/sdk\|@pika/sdk" --include="*.ts" --include="*.tsx" --exclude-dir=providers src/orchestrator src/ui; then
  echo "ERROR: Provider SDKs imported outside /providers directory"
  exit 1
fi
```

**Pre-commit hook** - catch before commit:

```bash
# .husky/pre-commit
npm run lint && npm run check-boundaries
```

---

### 2. Contract tests for adapters

Each adapter must pass the same test suite. This prevents "special snowflake" adapters.

**Shared contract test suite:**

```ts
// tests/contracts/TextToVideoProvider.contract.test.ts
export function createContractTests(
  providerName: string,
  createProvider: () => TextToVideoProvider
) {
  describe(`${providerName} - TextToVideoProvider contract`, () => {
    let provider: TextToVideoProvider

    beforeEach(() => {
      provider = createProvider()
    })

    it("returns normalized results with required fields", async () => {
      const result = await provider.generateVideo({
        prompt: "A cat walking",
        duration: 5
      })
      
      expect(result).toMatchObject({
        videoUrl: expect.any(String),
        status: "SUCCESS",
        providerId: expect.any(String),
        metadata: expect.any(Object)
      })
      expect(result.videoUrl).toMatch(/^https?:\/\//)
    })

    it("fails fast on timeout", async () => {
      const timeoutProvider = createProvider()
      jest.spyOn(timeoutProvider, 'generateVideo').mockImplementation(
        () => new Promise((_, reject) => 
          setTimeout(() => reject(new Error('Timeout')), 100)
        )
      )
      
      await expect(
        timeoutProvider.generateVideo({ prompt: "test" })
      ).rejects.toMatchObject({
        type: "PROVIDER_TIMEOUT"
      })
    })

    it("normalizes errors to standard format", async () => {
      const errorProvider = createProvider()
      jest.spyOn(errorProvider, 'generateVideo').mockRejectedValue(
        new Error("API rate limit exceeded")
      )
      
      await expect(
        errorProvider.generateVideo({ prompt: "test" })
      ).rejects.toMatchObject({
        type: expect.stringMatching(/PROVIDER_|RATE_LIMIT|ERROR/),
        message: expect.any(String)
      })
    })

    it("implements healthCheck", async () => {
      const health = await provider.healthCheck()
      expect(health).toMatchObject({
        status: expect.stringMatching(/HEALTHY|UNHEALTHY|DEGRADED/),
        latency: expect.any(Number)
      })
    })

    it("implements capabilities", () => {
      const caps = provider.capabilities()
      expect(caps).toMatchObject({
        maxDuration: expect.any(Number),
        supportedFormats: expect.any(Array),
        features: expect.any(Array)
      })
    })
  })
}
```

**Usage in each adapter:**

```ts
// providers/google-veo/GoogleVeoAdapter.test.ts
import { createContractTests } from '../../tests/contracts/TextToVideoProvider.contract.test'

describe('GoogleVeoAdapter', () => {
  createContractTests('GoogleVeo', () => new GoogleVeoAdapter(config))
  
  // Adapter-specific tests here
})
```

**CI enforcement:** All adapters must pass contract tests or build fails.

---

### 3. Orchestrator black-box tests

You test behavior, not providers.

```ts
it("falls back when provider fails", async () => {
  mockProviderFailure();
  const result = await generateVideo(request);
  expect(result.status).toBe("FAILED");
});
```

No provider SDKs in these tests. Ever.

---

### 4. Build-time boundaries

Use folder-level rules to enforce architectural boundaries.

**Directory structure:**

```
src/
  providers/          # ✅ May import SDKs
    google-veo/
    runway/
    pika/
  orchestrator/       # ❌ Cannot import SDKs or providers directly
  ui/                 # ❌ Cannot import SDKs or providers directly
  shared/             # ✅ Shared types and interfaces only
    types/
    interfaces/
```

**Nx module boundaries** (if using Nx):

```json
// nx.json
{
  "enforceModuleBoundaries": [
    {
      "sourceTag": "scope:orchestrator",
      "onlyDependOnLibsWithTags": ["scope:shared"],
      "bannedExternalImports": ["@google/genai", "@runwayml/sdk"]
    },
    {
      "sourceTag": "scope:ui",
      "onlyDependOnLibsWithTags": ["scope:shared", "scope:orchestrator"],
      "bannedExternalImports": ["@google/genai", "@runwayml/sdk"]
    },
    {
      "sourceTag": "scope:providers",
      "onlyDependOnLibsWithTags": ["scope:shared"]
    }
  ]
}
```

**TypeScript project references** (monorepo):

```json
// tsconfig.base.json
{
  "compilerOptions": {
    "composite": true,
    "paths": {
      "@providers/*": ["packages/providers/*/src"],
      "@orchestrator/*": ["packages/orchestrator/src"],
      "@shared/*": ["packages/shared/src"]
    }
  },
  "references": [
    { "path": "./packages/shared" },
    { "path": "./packages/orchestrator" },
    { "path": "./packages/providers/google-veo" }
  ]
}
```

**Dependency-cruiser** - visualize and enforce boundaries:

```js
// .dependency-cruiser.js
module.exports = {
  forbidden: [
    {
      name: 'no-sdk-in-orchestrator',
      severity: 'error',
      from: { path: '^src/orchestrator' },
      to: { path: '@google/genai|@runwayml/sdk|@pika/sdk' }
    },
    {
      name: 'no-sdk-in-ui',
      severity: 'error',
      from: { path: '^src/ui' },
      to: { path: '@google/genai|@runwayml/sdk|@pika/sdk' }
    },
    {
      name: 'no-provider-imports-in-orchestrator',
      severity: 'error',
      from: { path: '^src/orchestrator' },
      to: { path: '^src/providers' }
    }
  ]
}
```

---

### 5. Runtime enforcement via events

Every adapter must emit events. If an adapter skips events, it's broken by definition.

**Event schema contract:**

```ts
// shared/types/events.ts
export interface VideoRequestedEvent {
  type: 'video.requested'
  requestId: string
  prompt: string
  timestamp: number
  userId?: string
}

export interface VideoProviderSelectedEvent {
  type: 'video.provider.selected'
  requestId: string
  providerId: string
  reason: string
  timestamp: number
}

export interface VideoGeneratedEvent {
  type: 'video.generated'
  requestId: string
  providerId: string
  videoUrl: string
  duration: number
  metadata: Record<string, unknown>
  timestamp: number
}

export interface VideoFailedEvent {
  type: 'video.failed'
  requestId: string
  providerId: string
  error: {
    type: string
    message: string
    code?: string
  }
  timestamp: number
}
```

**Event emitter interface (required for all adapters):**

```ts
// shared/interfaces/EventEmitter.ts
export interface EventEmitter {
  emit(event: VideoRequestedEvent | VideoProviderSelectedEvent | 
       VideoGeneratedEvent | VideoFailedEvent): void
}
```

**Adapter implementation check:**

```ts
// tests/contracts/EventEmitter.contract.test.ts
it('emits video.requested before generation', async () => {
  const events: Event[] = []
  const provider = new GoogleVeoAdapter({ 
    eventEmitter: { emit: (e) => events.push(e) }
  })
  
  await provider.generateVideo({ prompt: "test" })
  
  expect(events).toContainEqual(
    expect.objectContaining({ type: 'video.requested' })
  )
})

it('emits video.generated on success', async () => {
  const events: Event[] = []
  const provider = new GoogleVeoAdapter({ 
    eventEmitter: { emit: (e) => events.push(e) }
  })
  
  await provider.generateVideo({ prompt: "test" })
  
  expect(events).toContainEqual(
    expect.objectContaining({ 
      type: 'video.generated',
      videoUrl: expect.any(String)
    })
  )
})
```

**Runtime monitoring** - alert if events are missing:

```ts
// orchestrator/monitoring/eventValidator.ts
export function validateEventFlow(requestId: string, events: Event[]) {
  const required = ['video.requested', 'video.provider.selected']
  const hasRequired = required.every(type => 
    events.some(e => e.type === type && e.requestId === requestId)
  )
  
  if (!hasRequired) {
    alert('Missing required events for request', { requestId, events })
  }
}
```

---

### 6. CI/CD pipeline enforcement

Automate all checks in your CI pipeline. Fail fast, fail loudly.

**GitHub Actions example:**

```yaml
# .github/workflows/architecture-checks.yml
name: Architecture Enforcement

on: [pull_request]

jobs:
  check-boundaries:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Check dependency boundaries
        run: npm run check-boundaries
      - name: Run ESLint
        run: npm run lint
      - name: Run contract tests
        run: npm run test:contracts
      - name: Check for SDK imports
        run: |
          if grep -r "@google/genai\|@runwayml/sdk" --exclude-dir=providers src/; then
            echo "❌ SDK imports found outside /providers"
            exit 1
          fi
```

**Pre-merge checklist** (enforce in PR template):

```markdown
## Architecture Checklist

- [ ] No SDK imports outside `/providers/*`
- [ ] New adapter passes all contract tests
- [ ] Orchestrator tests don't import provider SDKs
- [ ] All adapters emit required events
- [ ] No provider-specific types in orchestrator
```

### 7. Code review enforcement

**Required reviewers:**
- Architecture owner must approve any changes to `/orchestrator` or `/providers`
- Changes to boundaries require explicit approval

**Review checklist:**
1. Does this PR import any provider SDKs outside `/providers`?
2. Does this adapter implement the full contract interface?
3. Are all required events being emitted?
4. Does the orchestrator remain provider-agnostic?

### 8. Optional frameworks that help

Use these **only if needed**:

* **Temporal** for long-running video jobs
* **BullMQ** or **Cloud Tasks** for async execution
* **OpenTelemetry** for traces and health
* **Pact** if providers become internal services later

Do not add these early.

---

## The hard truth

If:

* The UI imports SDKs
* The dashboard is queried at runtime
* One service knows all providers

Then you built a distributed monolith.

If you want, next we can:

* Design the exact event schema
* Pressure-test this against serverless
* Or map this onto your existing Node front end step by step

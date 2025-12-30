```markdown

Followed with **how you actually enforce this**, because patterns without enforcement rot fast.

---

````md
# Text-to-API Service Architecture

## Goal

Provide a stable, testable, and provider-agnostic way to invoke external APIs from text prompts.

This system must:
- Avoid direct SDK usage in UI code
- Isolate vendor churn
- Support multiple providers (APIs)
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
export interface TextToApiProvider {
  invoke(input: ApiPrompt): Promise<ApiResult>;
  healthCheck(): Promise<HealthStatus>;
  capabilities(): ProviderCapabilities;
}
```

### Example Adapters

* `GoogleApiAdapter`
* `ProviderXAdapter`
* `InternalServiceAdapter`

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

1. Client calls `POST /invoke`
2. Orchestrator emits `RequestSubmitted`
3. Provider is selected
4. Adapter executes request
5. Orchestrator emits `RequestSucceeded` or `RequestFailed`
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

* No shared "API SDK service" that becomes a monolith
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

---

## Now the important part: **how you enforce this**

Patterns don’t survive by documentation.  
They survive by **friction and tests**.

### 1. Dependency enforcement (this is non-negotiable)

**Rule:**  
Front-end and orchestration code must not import provider SDKs.

**How to enforce**
- ESLint `no-restricted-imports`

```js
no-restricted-imports: [
  "error",
  {
    "paths": [
      {
        "name": "@google/api-sdk",
        "message": "Import SDKs only inside provider adapters"
      }
    ]
  }
]
```

This catches violations at PR time.

---

### 2. Contract tests for adapters

Each adapter must pass the same test suite.

```ts
describe("TextToApiProvider contract", () => {
  it("returns normalized results", async () => {
    const result = await provider.invoke(prompt);
    expect(result.output).toBeDefined();
  });

  it("fails fast on timeout", async () => {
    expect(provider.invoke(badPrompt)).rejects.toMatchObject({
      type: "PROVIDER_TIMEOUT"
    });
  });
});
```

This prevents “special snowflake” adapters.

---

### 3. Orchestrator black-box tests

You test behavior, not providers.

```ts
it("falls back when provider fails", async () => {
  mockProviderFailure();
  const result = await invoke(request);
  expect(result.status).toBe("FAILED");
});
```

No provider SDKs in these tests. Ever.

---

### 4. Build-time boundaries

Use folder-level rules.

* `/providers` may import SDKs
* `/orchestrator` may not
* `/ui` may not

Tools:

* ESLint
* Nx module boundaries
* TypeScript project references

---

### 5. Runtime enforcement via events

Every adapter emits events.

Examples:

* `request.submitted`
* `request.provider.selected`
* `request.succeeded`
* `request.failed`

If an adapter skips events, it’s broken by definition.

---

### 6. Optional frameworks that help

Use these **only if needed**:

* **Temporal** for long-running jobs
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

````

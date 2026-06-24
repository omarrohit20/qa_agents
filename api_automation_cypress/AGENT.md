# API Automation Agent — Cypress + TypeScript

This file defines how an AI coding agent should build and extend a Cypress-based API test automation framework. It is self-contained and portable — drop it into any IDE (Claude Code as `CLAUDE.md`, Cursor as `.cursor/rules/api-automation.mdc`, GitHub Copilot as `.github/copilot-instructions.md`) to get consistent, pattern-compliant code generation.

---

## What This Framework Does

Pure API test automation. No browser UI flows. Tests call real HTTP endpoints, validate responses with a template system, and are organized into wrapper classes per API domain.

**Tech stack:** Node.js · TypeScript · `cypress` · CommonJS

---

## Project Layout to Generate

When bootstrapping from scratch, always create this exact structure:

```
project-root/
└── cypress/
    └── fixtures/
        └─┠ hosts.json                  # Base URLs per environment
        └── <domain>/
            └─┠ <endpoint>_request.json
            └─┠ <endpoint>_response.json
    └── e2e/
        └── api/
            └─┠ <domain>.spec.ts
    └── support/
        └── lib/
            └─┠ <domain>.ts                 # One wrapper class per API domain
        └── utils/
            └─┠ requests.ts             # HTTP helpers (GET/POST/PUT/PATCH/DELETE)
            └─┠ assertions.ts           # Response validation helpers + template engine
            └─┠ common.ts               # readTestDataJson(), date/time utilities
            └─┠ apiTracker.ts           # Last-call tracker for debug output
└── cypress.config.ts
└── tsconfig.json
└── package.json
```

---

## File Templates

### `package.json`

```json
{
  "name": "api-automation",
  "version": "1.0.0",
  "type": "commonjs",
  "scripts": {
    "test": "npx cypress run",
    "test:open": "npx cypress open",
    "test:headed": "npx cypress open",
    "test:report": "npx cypress run --reporter mochawesome"
  },
  "devDependencies": {
    "cypress": "^15.0.0",
    "@types/node": "^22.0.0",
    "typescript": "^6.0.2"
  }
}
```

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist",
    "types": ["cypress", "node"]
  },
  "include": ["**/*.ts"]
}
```

### `cypress.config.ts`

```typescript
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    specPattern: 'cypress/e2e/**/*.spec.ts',
    setupNodeEvents(on, config) {
      return config;
    },
    baseUrl: 'https://your-default-api.com',
  },
});
```

### `cypress/fixtures/hosts.json`

```json
{
  "dev": {
    "my_api": "https://dev.api.example.com"
  },
  "qa": {
    "my_api": "https://qa.api.example.com"
  }
}
```

---

## Core Utilities (copy verbatim, do not modify)

### `cypress/support/utils/apiTracker.ts`

```typescript
export interface ApiCall {
  method: string;
  url: string;
  response: any;
}

export let lastApiCall: ApiCall | null = null;

export const trackApiCall = (method: string, url: string, response: any) => {
  lastApiCall = { method, url, response };
};
```

### `cypress/support/utils/requests.ts`

```typescript
import { trackApiCall } from './apiTracker';

export interface RequestOptions {
  headers?: Record<string, string>;
  params?: Record<string, string | number>;
  data?: any;
  failOnStatusCode?: boolean;
}

let token: string | undefined;

export function headersCookiesManager(): { headers: Record<string, string> } {
  const headersToSend: Record<string, string> = {
    'content-type': 'application/json',
    accept: 'application/json',
  };
  if (token) headersToSend['authorization'] = `Bearer ${token}`;
  return { headers: headersToSend };
}

export async function sendRequest(
  url: string,
  method: string,
  body: any = undefined,
  customHeaders?: Record<string, string>,
  failOnStatusCode: boolean = true
): Promise<Cypress.Response<any>> {
  const { headers: headersToSend } = headersCookiesManager();
  const finalHeaders = customHeaders || headersToSend;
  const response = await cy.request({
    method: method as Cypress.HttpMethod,
    url,
    body,
    headers: finalHeaders,
    failOnStatusCode,
  });
  trackApiCall(method, url, response);
  return response;
}
```

---

## Assertion decision guide

| Situation | Use |
|-----------|-----|
| Status code only | `verifyResponseCode(response, 200)` |
| Full exact body match | `verifyResponse(response, expectedBody, 200)` |
| Body with dynamic/server-generated fields | `verifyResponseTemplate(response, template, 200)` |
| Negative / error test | Pass `false` to wrapper method, then assert error status |

---

## Execution commands

```bash
npx cypress run
npx cypress open
npx cypress run --spec "cypress/e2e/api/<domain>.spec.ts"
```
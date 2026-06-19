# API Automation Agent — Playwright + TypeScript

This file defines how an AI coding agent should build and extend a Playwright-based API test automation framework. It is self-contained and portable — drop it into any IDE (Claude Code as `CLAUDE.md`, Cursor as `.cursor/rules/api-automation.mdc`, GitHub Copilot as `.github/copilot-instructions.md`) to get consistent, pattern-compliant code generation.

---

## What This Framework Does

Pure API test automation. No browser, no UI. Tests call real HTTP endpoints, validate responses with a template system, and are organized into wrapper classes per API domain.

**Tech stack:** Node.js · TypeScript · `@playwright/test` · CommonJS

---

## Project Layout to Generate

When bootstrapping from scratch, always create this exact structure:

```
project-root/
├── config/
│   └── hosts.json                  # Base URLs per environment
├── libs/
│   ├── <domain>.ts                 # One wrapper class per API domain
│   └── utils/
│       ├── requests.ts             # HTTP helpers (GET/POST/PUT/PATCH/DELETE)
│       ├── assertions.ts           # Response validation helpers + template engine
│       ├── common.ts               # readTestDataJson(), date/time utilities
│       └── apiTracker.ts           # Last-call tracker for debug output
├── test_data/
│   └── <domain>/
│       ├── <endpoint>_request.json
│       └── <endpoint>_response.json
├── spec/api/
│   └── <domain>.spec.ts
├── global-setup.ts
├── playwright.config.ts
├── tsconfig.json
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
    "test": "playwright test",
    "test:debug": "playwright test --debug",
    "test:headed": "playwright test --headed",
    "test:ui": "playwright test --ui",
    "test:report": "playwright show-report"
  },
  "devDependencies": {
    "@playwright/test": "^1.46.0",
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
    "types": ["@playwright/test", "node"]
  },
  "include": ["**/*.ts"]
}
```

### `playwright.config.ts`

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './spec',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  globalSetup: './global-setup',
  use: {
    baseURL: 'https://your-default-api.com',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
});
```

### `global-setup.ts`

```typescript
async function globalSetup() {
  console.log('🚀 Starting API test suite...');
}

export default globalSetup;
```

### `config/hosts.json`

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

### `libs/utils/apiTracker.ts`

```typescript
import { APIResponse } from '@playwright/test';

export interface ApiCall {
  method: string;
  url: string;
  response: APIResponse;
}

export let lastApiCall: ApiCall | null = null;

export const trackApiCall = (method: string, url: string, response: APIResponse) => {
  lastApiCall = { method, url, response };
};
```

### `libs/utils/requests.ts`

```typescript
import { APIRequestContext, APIResponse } from '@playwright/test';
import { trackApiCall } from './apiTracker';

let token: string | undefined;

export interface RequestOptions {
  headers?: Record<string, string>;
  params?: Record<string, string | number>;
  data?: any;
  failOnStatusCode?: boolean;
}

export function headersCookiesManager(): { headers: Record<string, string> } {
  const headersToSend: Record<string, string> = {
    'content-type': 'application/json',
    'accept': 'application/json',
  };
  if (token) headersToSend['authorization'] = `Bearer ${token}`;
  return { headers: headersToSend };
}

export async function sendRequest(
  context: APIRequestContext,
  method: string,
  url: string,
  params: any = {},
  customHeaders?: Record<string, string>,
  failOnStatusCode: boolean = true
): Promise<APIResponse> {
  const { headers: headersToSend } = headersCookiesManager();
  const finalHeaders = customHeaders || headersToSend;
  try {
    const response = await context.fetch(url, {
      method: method as 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH',
      headers: finalHeaders,
      ...params,
    });
    trackApiCall(method, url, response);
    if (!response.ok() && failOnStatusCode) {
      throw new Error(`HTTP ${response.status()}: ${await response.text()}`);
    }
    return response;
  } catch (error) {
    console.error(`Request failed: ${method} ${url}`, error);
    throw error;
  }
}

export async function sendGetRequest(
  context: APIRequestContext, url: string, headers?: Record<string, string>, failOnStatusCode = true
): Promise<APIResponse> {
  return sendRequest(context, 'GET', url, { failOnStatusCode }, headers, failOnStatusCode);
}

export async function sendPostRequest(
  context: APIRequestContext, url: string, json?: any, headers?: Record<string, string>, failOnStatusCode = true
): Promise<APIResponse> {
  const payload = json ? (typeof json === 'object' ? json : JSON.parse(json)) : undefined;
  return sendRequest(context, 'POST', url, { data: payload, failOnStatusCode }, headers, failOnStatusCode);
}

export async function sendPutRequest(
  context: APIRequestContext, url: string, json: any, headers?: Record<string, string>, failOnStatusCode = true
): Promise<APIResponse> {
  const payload = json ? (typeof json === 'object' ? json : JSON.parse(json)) : undefined;
  return sendRequest(context, 'PUT', url, { data: payload, failOnStatusCode }, headers, failOnStatusCode);
}

export async function sendPatchRequest(
  context: APIRequestContext, url: string, json: any, headers?: Record<string, string>, failOnStatusCode = true
): Promise<APIResponse> {
  const payload = json ? (typeof json === 'object' ? json : JSON.parse(json)) : undefined;
  return sendRequest(context, 'PATCH', url, { data: payload, failOnStatusCode }, headers, failOnStatusCode);
}

export async function sendDeleteRequest(
  context: APIRequestContext, url: string, headers?: Record<string, string>, failOnStatusCode = true
): Promise<APIResponse> {
  return sendRequest(context, 'DELETE', url, { failOnStatusCode }, headers, failOnStatusCode);
}
```

### `libs/utils/assertions.ts`

```typescript
import { APIResponse, expect } from '@playwright/test';
import { lastApiCall } from './apiTracker';

export function verifyResponseCode(response: APIResponse, expectedResponseCode: number): void {
  expect(response.status()).toBe(expectedResponseCode);
}

export function verifyResponseIsSuccessful(response: APIResponse): void {
  expect(response.status()).toBe(200);
}

export function verifyResponseIsSuccessfulCreateEntity(response: APIResponse): void {
  expect(response.status()).toBe(201);
}

export async function verifyResponse(
  response: APIResponse, expectedResponse: any, expectedResponseCode: number
): Promise<void> {
  verifyResponseCode(response, expectedResponseCode);
  const body = await response.json();
  expect(body).toEqual(expectedResponse);
}

export async function verifyResponseTemplate(
  response: APIResponse, expectedResponse: any, expectedResponseCode: number
): Promise<void> {
  try {
    verifyResponseCode(response, expectedResponseCode);
    const body = await response.json();
    verifyResponseMatchExpected(body, expectedResponse);
  } catch (error) {
    console.error('❌ API ASSERTION FAILED');
    console.error(`METHOD: ${lastApiCall?.method ?? 'unknown'}`);
    console.error(`URL: ${lastApiCall?.url ?? 'unknown'}`);
    try { console.error(JSON.stringify(await response.json(), null, 2)); } catch {}
    throw error;
  }
}

export function verifyResponseMatchExpected(actual: unknown, expected: unknown): void {
  if (Array.isArray(actual) && Array.isArray(expected)) {
    actual.forEach((a, i) => compareResponseHashes(a, expected[i]));
    expected.forEach((e, i) => compareResponseHashes(actual[i], e));
  } else {
    compareResponseHashes(actual, expected);
  }
}

function compareResponseHashes(actual: unknown, expected: unknown, path = 'root'): void {
  if (expected === 'skip') return;

  if (typeof actual !== 'object' || actual === null || typeof expected !== 'object' || expected === null) {
    compareValues(actual, expected, path);
    return;
  }

  if (Array.isArray(actual) && Array.isArray(expected)) {
    if (actual.length !== expected.length) {
      throw new Error(`[SCHEMA MISMATCH] ARRAY LENGTH MISMATCH at ${path}. Expected ${expected.length}, got ${actual.length}`);
    }
    actual.forEach((item, i) => compareResponseHashes(item, expected[i], `${path}[${i}]`));
    return;
  }

  const actualObj = actual as Record<string, unknown>;
  const expectedObj = expected as Record<string, unknown>;

  Object.keys(expectedObj).forEach((key) => {
    if (!(key in actualObj)) {
      throw new Error(`[SCHEMA MISMATCH]\nPath: ${path}.${key}\nProblem: Missing key in response\nExpected: ${JSON.stringify(expectedObj[key])}`);
    }
  });

  Object.keys(expectedObj).forEach((key) => {
    compareResponseHashes(actualObj[key], expectedObj[key], `${path}.${key}`);
  });
}

function compareValues(actual: unknown, expected: unknown, path: string): void {
  if (typeof expected === 'string') {
    handleExpectedString(actual, expected, path);
    return;
  }
  if (JSON.stringify(actual) !== JSON.stringify(expected)) {
    throw new Error(`[VALUE MISMATCH]\nPath: ${path}\nExpected: ${JSON.stringify(expected)}\nActual: ${JSON.stringify(actual)}`);
  }
}

function handleExpectedString(actual: unknown, expected: string, path: string): void {
  const actualValue = String(actual);
  if (expected.includes('match_regex')) {
    const match = expected.match(/\/(.+)\//);
    if (!match) throw new Error(`Invalid regex at ${path}`);
    expect(actualValue).toMatch(new RegExp(match[1]));
    return;
  }
  switch (expected) {
    case 'only_digits':    expect(actualValue).toMatch(/^\d+$/); break;
    case 'only_chars':     expect(actualValue).toMatch(/^[a-zA-Z]+$/); break;
    case 'should_not_be_null': expect(actual).not.toBeNull(); break;
    case 'skip':           break;
    default:               expect(actual).toEqual(expected);
  }
}
```

### `libs/utils/common.ts`

```typescript
import * as fs from 'fs';
import * as path from 'path';

export function readTestDataJson(filename: string): any {
  const fixturePath = path.join(__dirname, '../../test_data', filename);
  return JSON.parse(fs.readFileSync(fixturePath, 'utf-8'));
}

export function createTimestamp(): number { return Math.floor(Date.now() / 1000); }
export function timeInIso(): string { return new Date().toISOString(); }
export function timeInIsoShift(days: number): string {
  const d = new Date(); d.setDate(d.getDate() + days); return d.toISOString();
}
export function dateShiftDays(daysNum: number): Date {
  const d = new Date(); d.setDate(d.getDate() + daysNum); return d;
}
export function convertToJson(object: any): any {
  return typeof object === 'string' ? JSON.parse(object) : object;
}
```

---

## Pattern: API Wrapper Class

One class per API domain. File: `libs/<domain>.ts`

**Rules:**
- Constructor accepts an optional `APIRequestContext` and optional `baseUrl`
- If no context provided, call `this.initContext()` lazily and await via `this.contextReady`
- Always call `await this.ensureContext()` at the top of each method
- Load all fixture files in `initVariables()`, called from the constructor
- Store request/response templates as public class properties
- Auth tokens are stored on the instance; `initContext()` re-runs after auth to inject the cookie

```typescript
import { APIRequestContext, request, APIResponse } from '@playwright/test';
import { readTestDataJson } from './utils/common';
import { sendGetRequest, sendPostRequest, sendPutRequest, sendDeleteRequest } from './utils/requests';

export class MyDomain {
  private baseUrl: string = '';
  private context?: APIRequestContext;
  private contextReady?: Promise<void>;
  private authToken?: string;

  // Public template properties — one pair per endpoint
  public createRequest: any;
  public createResponse: any;
  public updateRequest: any;
  public updateResponse: any;

  constructor(context?: APIRequestContext, baseUrl?: string) {
    const hostConfig = require('../config/hosts.json');
    const env = process.env.ENV || 'dev';
    this.baseUrl = baseUrl ?? hostConfig[env]?.my_domain ?? '';

    if (context) {
      this.context = context;
    } else {
      this.contextReady = this.initContext();
    }

    this.initVariables();
  }

  async initContext(): Promise<void> {
    const headers: Record<string, string> = {
      'accept': 'application/json',
      'content-type': 'application/json',
    };
    if (this.authToken) headers['Cookie'] = `token=${this.authToken}`;
    this.context = await request.newContext({ baseURL: this.baseUrl, extraHTTPHeaders: headers });
  }

  private async ensureContext(): Promise<void> {
    if (this.context) return;
    if (!this.contextReady) throw new Error('API request context was not initialized.');
    await this.contextReady;
    if (!this.context) throw new Error('Failed to initialize API request context.');
  }

  // --- Endpoint methods ---

  async getList(failOnStatusCode = true): Promise<APIResponse> {
    await this.ensureContext();
    return sendGetRequest(this.context!, `${this.baseUrl}/items`, undefined, failOnStatusCode);
  }

  async getById(id: string, failOnStatusCode = true): Promise<APIResponse> {
    await this.ensureContext();
    return sendGetRequest(this.context!, `${this.baseUrl}/items/${id}`, undefined, failOnStatusCode);
  }

  async create(payload: any, failOnStatusCode = true): Promise<APIResponse> {
    await this.ensureContext();
    return sendPostRequest(this.context!, `${this.baseUrl}/items`, payload, undefined, failOnStatusCode);
  }

  async update(id: string, payload: any, failOnStatusCode = true): Promise<APIResponse> {
    await this.ensureContext();
    return sendPutRequest(this.context!, `${this.baseUrl}/items/${id}`, payload, undefined, failOnStatusCode);
  }

  async delete(id: string, failOnStatusCode = true): Promise<APIResponse> {
    await this.ensureContext();
    return sendDeleteRequest(this.context!, `${this.baseUrl}/items/${id}`, undefined, failOnStatusCode);
  }

  // --- Auth helper (only if the API requires token-based auth) ---

  async authenticate(username: string, password: string, failOnStatusCode = true): Promise<APIResponse> {
    await this.ensureContext();
    const response = await sendPostRequest(this.context!, `${this.baseUrl}/auth`, { username, password }, undefined, failOnStatusCode);
    if (response.status() === 200) {
      const body = await response.json();
      if (body?.token) { this.authToken = body.token; await this.initContext(); }
    }
    return response;
  }

  getAuthToken(): string | undefined { return this.authToken; }

  private initVariables(): void {
    this.createRequest  = readTestDataJson('my_domain/create_request.json');
    this.createResponse = readTestDataJson('my_domain/create_response.json');
    this.updateRequest  = readTestDataJson('my_domain/update_request.json');
    this.updateResponse = readTestDataJson('my_domain/update_response.json');
  }
}
```

---

## Pattern: Test Data Files

Two JSON files per endpoint: one for the request body, one for the expected response.

**`test_data/<domain>/create_request.json`** — literal payload sent to the API:
```json
{
  "name": "Test Item",
  "price": 99,
  "active": true
}
```

**`test_data/<domain>/create_response.json`** — response template with validation keywords:
```json
{
  "id": "skip",
  "name": "Test Item",
  "price": 99,
  "active": true,
  "createdAt": "should_not_be_null",
  "slug": "only_chars",
  "code": "only_digits",
  "uuid": "match_regex:/^[0-9a-f-]{36}$/"
}
```

### Validation keywords (use in response templates only)

| Keyword | Meaning |
|---------|---------|
| `"skip"` | Ignore this field completely — do not assert |
| `"should_not_be_null"` | Field must exist and not be null |
| `"only_chars"` | Value must match `/^[a-zA-Z]+$/` |
| `"only_digits"` | Value must match `/^\d+$/` |
| `"match_regex:/pattern/"` | Value must match the given regex |
| any literal value | Exact equality check |

**Rules for templates:**
- Omit a key from the template if you don't want it validated at all (unlike `"skip"` which explicitly acknowledges it exists)
- Templates can be nested objects and arrays
- Use `"skip"` for IDs, timestamps, and any server-generated values you cannot predict

---

## Pattern: Spec File

```typescript
import { test, expect } from '@playwright/test';
import { MyDomain } from '../../libs/my_domain';
import {
  verifyResponseCode,
  verifyResponseIsSuccessful,
  verifyResponseTemplate,
} from '../../libs/utils/assertions';

let api: MyDomain;

test.describe('MyDomain API', () => {

  test.beforeAll(async () => {
    api = new MyDomain();
    // If auth is required:
    // const authResp = await api.authenticate(api.authRequest.username, api.authRequest.password);
    // verifyResponseCode(authResp, 200);
  });

  test('GET list returns array', async () => {
    const response = await api.getList();
    verifyResponseIsSuccessful(response);
    const body = await response.json();
    expect(Array.isArray(body)).toBe(true);
  });

  test('POST create returns expected shape', async () => {
    const response = await api.create(api.createRequest);
    verifyResponseTemplate(response, api.createResponse, 200);
  });

  test('PUT update returns updated values', async () => {
    const createResp = await api.create(api.createRequest);
    verifyResponseCode(createResp, 200);
    const id = String((await createResp.json()).id);

    const updateResp = await api.update(id, api.updateRequest);
    verifyResponseTemplate(updateResp, api.updateResponse, 200);
  });

  test('DELETE removes the resource', async () => {
    const createResp = await api.create(api.createRequest);
    const id = String((await createResp.json()).id);

    const deleteResp = await api.delete(id);
    verifyResponseCode(deleteResp, 200);
  });

  test('negative: GET non-existent ID returns 404', async () => {
    const response = await api.getById('nonexistent-id-99999', false);
    verifyResponseCode(response, 404);
  });

});
```

---

## Assertion Decision Guide

| Situation | Use |
|-----------|-----|
| Just check the status code | `verifyResponseCode(response, 200)` |
| Full exact match of response body | `verifyResponse(response, expectedBody, 200)` |
| Response with dynamic/server-generated fields | `verifyResponseTemplate(response, template, 200)` — put `"skip"` or `"should_not_be_null"` for dynamic fields |
| Check a specific field directly | `const body = await response.json(); expect(body.field).toBe(value)` |
| Negative test (expect error) | Pass `false` as the last arg to the wrapper method, then assert the error status code |

---

## Environment Configuration

- Base URLs live in `config/hosts.json` keyed by environment name
- Select environment: `ENV=qa npx playwright test`
- Default is `dev` when `ENV` is not set
- Add new environments by adding a new key to `hosts.json`
- Add new API base URLs as new keys under each environment

---

## Adding a New API Domain: Checklist

1. Add the base URL to every environment in `config/hosts.json`
2. Create `libs/<domain>.ts` using the wrapper class pattern above
3. Create `test_data/<domain>/` and add request/response JSON fixtures
4. Create `spec/api/<domain>.spec.ts` using the spec pattern above
5. Run: `npx playwright test spec/api/<domain>.spec.ts`

---

## Running Tests

```bash
# All tests
npx playwright test

# One spec file
npx playwright test spec/api/booking.spec.ts

# One test by name
npx playwright test --grep "Create a new booking"

# One browser only
npx playwright test --project=chromium

# Debug with step-through
npx playwright test --debug

# Interactive UI
npx playwright test --ui

# Specific environment
ENV=qa npx playwright test
```

---

## Installation Instructions per IDE

### Claude Code
Save as `CLAUDE.md` at the repository root. Claude Code automatically loads it at the start of every session.

### Cursor
Save as `.cursor/rules/api-automation.mdc` at the repository root. Cursor loads `.mdc` files from `.cursor/rules/` as always-on context rules.

### GitHub Copilot (VS Code)
Save as `.github/copilot-instructions.md` at the repository root. Copilot picks it up automatically for all completions in the workspace.

### Standalone / sharing
This file is fully self-contained. Share it as-is. The recipient drops it into their IDE using whichever path applies above — no other files from this repo are required to use it as a generation guide.

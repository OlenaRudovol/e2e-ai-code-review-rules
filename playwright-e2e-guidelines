# Playwright E2E Testing Guidelines

## Design Principles

These rules are intentionally strict to enable reliable AI-assisted code review. They favor deterministic signals, explicit synchronization, and clear intent over convenience or visual feedback.
If a pattern is ambiguous for AI to reason about, it is considered invalid.

## ✅ Required Patterns

### Dependency Injection & Fixtures

- **Use Fixtures**: Never instantiate page objects or helpers manually in tests.
Inject everything via Playwright fixtures:

  ```typescript
  // ✅ Good: Dependencies are injected;
  test('Submit form', async ({ loginPage, dashboardPage }) => {...});
  
  // ❌ Bad: Manual instantiation;
  test('Submit form', async ({ page }) => {
    const loginPage = new LoginPage(page); 
  });
  ```

### Test Structure

- **Test Organization**: Every test must use a top-level `test.describe` block to group related tests:

  ```typescript 
  test.describe('Feature: Resource Management', () => {
    test('Create new resource', async ({ ... }) => {
      // Test code
    });
  });
  ```
  
- **User Context**: Every test.describe block that needs authentication must set the authenticated user context:

  ```typescript  
  test.describe('Feature: Resource Management', () => {
    test.use({ authenticatedUser: users.admin });
    // ...
  });
  
  ```

- **Step Documentation**: Every UI/API action sequence must be wrapped in test.step(). Maximum 4 actions per step. Label must describe user intent:

  ```typescript
  await test.step('Fill and submit the registration form', async () => {
    await registrationPage.fillDetails(userData);
    await registrationPage.submit();
  });
  ```

- **Categorization**: Every test must have tag e.g. @Smoke, @FeatureX. See our separate [Tag Policy](https://playwright.dev/docs/best-practices) guide.
Why: This allows for targeted execution in CI/CD.

  ```typescript
  test('…', { tag: ['@Smoke', '@FeatureX'] }, async () => { … });
  ```

- **Test isolation**: Tests must be completely independent and atomic. No test should depend on the state, data, or side effects created by another test. Ensure data-level isolation - create/verify/clean your own data via fixtures or API:

  ```typescript 
  // ✅ Good: self-contained
  test('Create & verify resource', async ({ apiHelpers, authenticatedUser }) => {
    const id = await apiHelpers.createResource(authenticatedUser, { name });
    // Test code
    await apiHelpers.deleteResource(id); // or rely on global cleanup
  });

  // ❌ Bad: cross-test dependency
  test('Edit resource from previous test', async () => {
    await page.getByText('Resource left by previous test').click();
  });
  ```

### Test Data Management

- **Unique Test Data**: Every created entity name/title must contain a unique suffix (${Date.now()} or uuid) identifier.
Why: To avoid collisions between parallel or repeated test runs.

  ```typescript
  const resourceName = `Test Resource ${Date.now()}`;
  ```

- **UI Data Creation**: UI creation methods must use @registerForCleanup() decorator:

  ```typescript
  @registerForCleanup()
  public async createResource(name: string): Promise<number> {
    // Creation code
  }
  ```

- **API Data Creation**: API creation must push ID to global.testDataRegistry (global cleanup):
  ```typescript
  const resourceId = await apiClient.createResource(data);
  global.testDataRegistry.push(resourceId);
  ```

### Synchronization & Stability

- **UI Operations**: UI actions that trigger requests must use Promise.all([waitForResponse, action]).
Why: Properly handle network calls and parallel operations.

  ```typescript
  await Promise.all([
    page.waitForResponse(resp => resp.url().includes('/api/save')),
    formPage.clickSaveButton(),
  ]);
  ```
  
- **API Operations**: Execute independent API operations in parallel for better performance:

  ```typescript
  await Promise.all([
    apiClient.createResource(data1),
    apiClient.createResource(data2),
  ]);
  ```

- **Search Index Sync**: After data creation (UI or API), always wait for search indexing:

  ```typescript
  await searchHelper.waitForIndexing([resourceName]);
  ```

- **Background Job Verification**: When operations trigger background jobs, always wait for job completion.
  Why: Required for eventual consistency in async flows.

  ```typescript
  await listingPage.performBulkAction();
  await jobHelper.waitForJobCompletion(
    JobType.BULK_OPERATION,
    authenticatedUser
  );
  ```

## ❌ Prohibited Patterns

- **Relative Imports**: Do not use relative imports (../). Use path aliases only (@utils/…).

  ```typescript
  // ❌ Bad: import { Something } from '../../../utils/helpers';
  // ✅ Good: import { Something } from '@utils/helpers';
  ```

- **Wait Anti-patterns**: Never use fixed timeouts or unreliable wait helpers.

  ```typescript
    ❌ .waitForTimeout() // use locators/assertions instead
    ❌ .waitForLoadState('networkidle') // too unstable for modern SPAs
    ❌ .waitForSelector(selector) without {state: 'visible'} // use expect(locator).toBeVisible() instead
    ❌ forceWait() or sleep() // The ultimate "flaky test" culprits
  ```

- **Page Object & Locator Discipline**: All locators must be stored as private class properties in page objects.
Never use locators:
  -  directly in test files
  -  inline inside page object methods
 
  ```typescript 
  // ❌ Bad: direct in test
  await page.locator('[data-testid="save"]').click();
  // ❌ Bad: inline (even good locator type)
  async clickSave() {
    await this.page.getByRole('button', { name: 'Save' }).click();  // duplicated, hard to change centrally
  }

  // ✅ Good
  class LoginPage {
    private readonly saveButton = this.page.getByRole('button', { name: 'Save' });
    async clickSave() {
      await this.saveButton.click();
    }
  }
  ```

## 🧪 Test Verification Best Practices

- **API-First Approach**: Create all prerequisite data via API, never via UI unless testing UI creation itself.

  ```typescript
  // ✅ Good: Create prerequisite data via API, test workflow via UI
  const resourceId = await apiHelpers.createResource(authenticatedUser, testData);
  // Now test UI workflow with the created resource
  ```

- **Soft Assertions**: Use expect.soft(…) when verifying ≥3 related conditions in one step. Fail the test only after collecting all issues.

- **Notification Verification**: Never verify business logic outcome via transient UI notifications/toasts. Use API calls or stable DOM elements.
Why: transient notification messages may be unreliable.

  ```typescript
  // ✅ Good: Verify state changes via API or stable UI elements
  const status = await apiClient.getResourceStatus(resourceId);
  expect(status).toBe('completed');
  ```

## 📝 Naming Conventions

- **Test Names**: Start test names with a Ticket ID (if applicable) or a clear feature prefix:

  ```typescript
  // ✅ Good: test('TICKET-123 Verify resource creation with special characters', ...);
  // ❌ Bad: test('test creation', ...);
  ```

## 🎨 Code Quality

- **Always Await**: Never forget `await` for async operations - missing await causes race conditions and flaky tests.

  ```typescript
  // ❌ Bad: Missing await causes race conditions
  page.click(selector); // Returns Promise<void>, not void
  const result = apiClient.getData(); // Returns Promise<Data>, not Data

  // ✅ Good: Properly awaited
  await page.click(selector);
  const result = await apiClient.getData();
  ```

## 🔗 Resources

// Any internal documentation can be added here
- [Playwright Best Practices](https://playwright.dev/docs/best-practices) — official guide
- [Playwright Test Documentation](https://playwright.dev/docs/intro)
- [Authentication & Storage State Reuse](https://playwright.dev/docs/auth)
- [Fixtures & Dependency Injection](https://playwright.dev/docs/test-fixtures)
- [Browser Contexts & Isolation](https://playwright.dev/docs/browser-contexts)
- Community: BrowserStack "15 Best Practices for Playwright in 2026", DeviQA Playwright Guide (2025)

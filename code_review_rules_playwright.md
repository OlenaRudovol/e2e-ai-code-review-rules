# Playwright E2E Testing Guidelines

This checklist helps teams enforce consistent code quality standards in E2E tests using Playwright.

## ✅ Required Patterns

### Test isolation
Tests must be fully independent and atomic. No test should depend on the state, data, or side effects created by another test.

### Dependency Injection & Fixtures

- **Use Fixtures**: Avoid manual instantiation of Page Objects or Helpers inside tests. Use Playwright fixtures to inject dependencies:

  ```typescript
  // ✅ Good: Dependencies are injected;
  test('Submit form', async ({ loginPage, dashboardPage }) => {
    await loginPage.login();
    await dashboardPage.checkStatus();
  });
  
  // ❌ Bad: Manual instantiation;
  test('Submit form', async ({ page }) => {
    const loginPage = new LoginPage(page); 
    await loginPage.login();
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
  
- **User Context**: Each suite must define the authenticated user context:

  ```typescript  
  test.describe('Feature: Resource Management', () => {
    test.use({ authenticatedUser: users.admin });
    // ...
  });
  
  ```

- **Step Documentation**: Wrap actions in test.step(). Keep steps atomic (max 3-4 actions) and use descriptive, user-oriented labels:

  ```typescript
  await test.step('Fill and submit the registration form for ${userData.name}', async () => {
    await registrationPage.fillDetails(userData);
    await registrationPage.submit();
  });
  ```

- **Categorization**: Every test must have tags e.g. @Smoke, @Regression, @FeatureName. This allows for targeted execution in CI/CD. More details about tag policy is described in a the internal document (follow the inserted link):

  ```typescript
  test(
    'Create new resource',
    {
      tag: ['@Feature', '@Component', '@Smoke'],
    },
    async ({ ... }) => {
      // Test code
    }
  );
  ```

### Test Data Management

- **Unique Test Data**: Test data names must include a unique identifier to avoid collisions between parallel or repeated test runs:

  ```typescriptz
  const uniqueId = new Date().getMilliseconds().toString();
  ...
  const resourceName = `Test Resource ${uniqueId}`;
  ```

- **UI Data Creation**: Use cleanup decorator for UI-based data creation methods:

  ```typescript
  @registerForCleanup()
  public async createResource(name: string): Promise<number> {
    // Creation code
  }
  ```

- **API Data Creation**: When creating data via API, register for global cleanup:
  ```typescript
  const resourceId = await apiClient.createResource(data);
  global.testDataRegistry.push(resourceId);
  ```

### Synchronization & Stability

- **UI Operations**: Properly handle network calls and parallel operations when UI actions trigger network calls:

  ```typescript
  await Promise.all([
    page.waitForResponse(resp => resp.url().includes('/api/save')),
    page.click('[data-testid="save-button"]'),
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

- **Background Job Verification**: When operations trigger background jobs, always wait for job completion:

  ```typescript
  await listingPage.performBulkAction();
  await jobHelper.waitForJobCompletion(
    JobType.BULK_OPERATION,
    authenticatedUser
  );
  ```

## ❌ Prohibited Patterns

- **Relative Imports**: Do not use imports with `../` - use path aliases instead:

  ```typescript
  // ❌ Bad: import { Something } from '../../../utils/helpers';
  // ✅ Good: import { Something } from '@utils/helpers';
  ```

- **Wait Anti-patterns**: Never use these unreliable waiting mechanisms:
Explanation: you can add any built-in or custom methods that you want to prohibit or deprecate.
  - `waitForNetworkIdle`
  - `waitForFewSeconds`
  - `forceWait`
  - `waitForTimeout`

- **Page Object Methods**: Always use page object methods instead of direct locators in tests:

  ```typescript
  // ❌ Bad: Direct locators in tests
  await page.locator('[data-testid="save-button"]').click();

  // ✅ Good: Use page object methods
  await pageObject.clickSaveButton();
  ```

## 🧪 Test Verification Best Practices

- **API-First Approach**: Use API for data creation in test setup/teardown:

  ```typescript
  // ✅ Good: Create prerequisite data via API, test workflow via UI
  const resourceId = await apiHelpers.createResource(
    authenticatedUser,
    testData
  );
  // Now test UI workflow with the created resource
  ```

- **Notification Verification**: Do not rely on transient notification messages for test verification as they may be unreliable:

  ```typescript
  // ✅ Good: Verify state changes via API or stable UI elements
  const status = await apiClient.getResourceStatus(resourceId);
  expect(status).toBe('completed');
  ```

## 📝 Naming Conventions

- **Test Names**: Start test names with a Ticket ID (if applicable) or a clear Feature prefix: 

  ```typescript
  // ✅ Good: test('TICKET-123 Verify resource creation with special characters', ...);
  // ❌ Bad: test('test creation', ...);
  ```

## 🎨 Code Quality

- **Selector Organization**: Store all selectors as class properties in page objects, never inline in methods.

  ```typescript
  // ✅ Good:
  private readonly saveButton = '[data-testid="save-button"]';
  async clickSave() { await this.page.click(this.saveButton); }

  // ❌ Bad:
  async clickSave() { await this.page.click('[data-testid="save-button"]'); }
  ```

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

- [Playwright Best Practices](https://playwright.dev/docs/best-practices)
- [Playwright Documentation](https://playwright.dev/)

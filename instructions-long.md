# Playwright E2E Testing Guidelines

This checklist helps teams enforce consistent code quality standards in E2E tests using Playwright.

## ✅ Required Patterns

### Test Structure

- **Test Organization**: Every test file **must** use a `test.describe` block to group related tests.

  ```typescript
  test.describe('Feature name tests', () => {
    // Tests go here
  });
  ```

- **User Context**: Every `test.describe` block **must** include authentication context to specify test permissions.

  ```typescript
  test.describe('Feature tests', () => {
    test.use({ authenticatedUser: users.admin });
    // Tests go here
  });
  ```

- **Step Documentation**: All test actions **must** be wrapped in `test.step()` with descriptive labels. No more than 3 actions per step.

  ```typescript
  await test.step(`Create a resource named "${resourceName}"`, async () => {
    // Test actions
  });
  ```

- **Test Tags**: All test cases **must** use tags for filtering and categorization:

  ```typescript
  test(
    'TICKET-123 Feature functionality',
    {
      tag: ['@Feature', '@Component', '@Smoke'],
    },
    async ({ page }) => {
      // Test code
    }
  );
  ```

- **User Parameters**: Always use the authenticated user parameter for user context in tests.

  ```typescript
  test('Test case', async ({ authenticatedUser, apiHelpers }) => {
    await apiHelpers.createResource(authenticatedUser, resourceData);
  });
  ```

### Test Data Management

- **Unique Test Data**: Test data names **must** include a unique identifier parameter for uniqueness.

  ```typescript
  const resourceName = `Test Resource ${uniqueId}`;
  ```

- **UI Data Creation**: Use cleanup decorator for UI-based data creation methods.

  ```typescript
  @registerForCleanup()
  public async createResource(name: string): Promise<number> {
    // Creation code
  }
  ```

- **API Data Creation**: When creating data via API, register for cleanup:
  ```typescript
  const resourceId = await apiClient.createResource(data);
  global.testDataRegistry.push(resourceId);
  ```

### Synchronization & Stability

- **Search Index Sync**: After data creation (UI or API), **always** wait for search indexing.

  ```typescript
  await searchHelper.waitForIndexing([resourceName], PageId.MAIN_LISTING);
  ```

- **Network Operations**: Properly handle network calls and parallel operations for stability and performance.

  ```typescript
  // When UI actions trigger network calls
  await Promise.all([
    page.waitForResponse(resp => resp.url().includes('/api/save')),
    page.click('[data-testid="save-button"]'),
  ]);

  // Execute independent API operations in parallel for better performance
  await Promise.all([
    apiClient.createResource(data1),
    apiClient.createResource(data2),
  ]);
  ```

- **Background Job Verification**: When operations trigger background jobs, **always** wait for job completion.

  ```typescript
  await listingPage.performBulkAction();
  await jobHelper.waitForJobCompletion(
    JobType.BULK_OPERATION,
    authenticatedUser
  );
  ```

## ❌ Prohibited Patterns

- **Relative Imports**: Do not use imports with `../` - use path aliases instead.

  ```typescript
  // ❌ Bad: import { Something } from '../../../utils/helpers';
  // ✅ Good: import { Something } from '@utils/helpers';
  ```

- **Wait Anti-patterns**: Never use these unreliable waiting mechanisms:

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

- **API-First Approach**: Use API for test setup/teardown when possible; reserve UI for testing actual user workflows.

  ```typescript
  // ✅ Good: Create prerequisite data via API, test workflow via UI
  const resourceId = await apiHelpers.createResource(
    authenticatedUser,
    testData
  );
  // Now test UI workflow with the created resource
  ```

- **Notification Verification**: Do not rely on transient notification messages for test verification as they may be unreliable.

  ```typescript
  // ✅ Good: Verify state changes via API or stable UI elements
  const status = await apiClient.getResourceStatus(resourceId);
  expect(status).toBe('completed');
  ```

## 📝 Naming Conventions

- **Test Names**: Start test names with ticket ID and use descriptive names.

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

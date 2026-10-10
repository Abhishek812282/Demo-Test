Playwright Context — OrangeHRM Automation

1. Project Overview
   This project contains automated end-to-end tests for the OrangeHRM web application using Playwright.

Application
Application: OrangeHRM

Automation tool: Playwright

Language: TypeScript

Test framework: Playwright Test

Pattern: Page Object Model (POM)

Browser coverage: Chromium, Firefox, WebKit

Test type: UI / End-to-End automation

The goal is to create stable, maintainable, reusable, and readable automated tests.

2. Application Modules
   The automation suite should cover the following major modules:

Login
Valid login

Invalid username

Invalid password

Empty username/password

Logout

Session validation

Admin
User listing

Search users

Add user

Edit user

Delete/disable user

Assign user role

Validate user permissions

PIM / Employee Management
Employee listing

Search employee

Add employee

Edit employee

Delete employee

View employee details

Upload employee profile picture

Validate mandatory fields

Leave
View leave list

Apply for leave

Search leave records

Cancel leave

Approve leave

Reject leave

Validate leave dates

Validate leave type and required fields

Attendance
Employee attendance

Punch in

Punch out

View attendance records

Search attendance records

Recruitment
Candidate listing

Add candidate

Search candidate

Edit candidate

Delete candidate

Candidate status changes

My Info
View personal information

Edit personal information

Contact details

Emergency contacts

Dependents

Job information

3. Project Structure
   Use the following structure:

orangehrm-playwright/
│
├── tests/
│ ├── login/
│ ├── admin/
│ ├── pim/
│ ├── leave/
│ ├── attendance/
│ └── recruitment/
│
├── pages/
│ ├── LoginPage.ts
│ ├── DashboardPage.ts
│ ├── AdminPage.ts
│ ├── EmployeePage.ts
│ ├── LeavePage.ts
│ ├── AttendancePage.ts
│ └── RecruitmentPage.ts
│
├── fixtures/
│ └── testFixtures.ts
│
├── utils/
│ ├── testData.ts
│ ├── constants.ts
│ └── helpers.ts
│
├── test-data/
│ └── employees.json
│
├── playwright.config.ts
├── package.json
└── playwright_context.md

4. Page Object Model Rules
   All page-specific locators and actions should be maintained inside page object classes.

Tests should not contain large amounts of locator logic.

Example
export class LoginPage {
constructor(private page: Page) {}

readonly usernameInput = this.page.getByPlaceholder('Username');
readonly passwordInput = this.page.getByPlaceholder('Password');
readonly loginButton = this.page.getByRole('button', { name: 'Login' });

async login(username: string, password: string) {
await this.usernameInput.fill(username);
await this.passwordInput.fill(password);
await this.loginButton.click();
}
}

Test:

test('valid login', async ({ page }) => {
const loginPage = new LoginPage(page);

await loginPage.login(
process.env.ORANGEHRM_USERNAME!,
process.env.ORANGEHRM_PASSWORD!
);

await expect(page).toHaveURL(/dashboard/);
});

5. Locator Strategy
   Use Playwright's recommended locator strategy.

Preferred order:

getByRole()

getByLabel()

getByPlaceholder()

getByText()

getByTestId()

CSS selectors only when necessary

XPath should generally be avoided

Good
page.getByRole('button', { name: 'Login' });

page.getByPlaceholder('Username');

Avoid
page.locator('//button[@type="submit"]');

Avoid brittle selectors based on generated CSS classes or DOM hierarchy.

6. Test Design
   Each test should:

Have one clear purpose

Use meaningful test names

Be independent where practical

Avoid dependency on test execution order

Use reusable page objects

Use explicit assertions

Clean up created test data when appropriate

Example:

test('should create a new employee successfully', async ({ page }) => {
// Arrange
// Act
// Assert
});

7. Assertions
   Every important action should have an appropriate assertion.

Do not consider a test successful merely because a click completed.

Example:

await loginPage.login(username, password);

await expect(page.getByText('Dashboard')).toBeVisible();

For successful employee creation:

await expect(
page.getByText('Successfully Saved')
).toBeVisible();

Assertions should verify the actual business result.

8. Test Data
   Do not hard-code credentials directly inside test files.

Use environment variables.

Example .env:

ORANGEHRM_BASE_URL=https://your-orangehrm-instance.example
ORANGEHRM_USERNAME=Admin
ORANGEHRM_PASSWORD=your-password

Access values using:

process.env.ORANGEHRM_USERNAME
process.env.ORANGEHRM_PASSWORD

Never commit real passwords, API keys, or other secrets to source control.

9. Playwright Configuration
   The configuration should include:

Base URL

Browser projects

Trace on failure/retry

Screenshot on failure

Video when appropriate

Reasonable timeout

HTML reporter

Example:

import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
testDir: './tests',

use: {
baseURL: process.env.ORANGEHRM_BASE_URL,
trace: 'retain-on-failure',
screenshot: 'only-on-failure',
video: 'retain-on-failure',
},

reporter: [['html', { open: 'never' }]],

projects: [
{
name: 'chromium',
use: { ...devices['Desktop Chrome'] },
},
{
name: 'firefox',
use: { ...devices['Desktop Firefox'] },
},
],
});

10. Authentication
    Where appropriate, use Playwright's authentication state feature to avoid logging in before every test.

Example:

await page.context().storageState({
path: 'playwright/.auth/user.json',
});

Do not reuse authentication state when the test specifically validates login functionality.

Login tests should independently test:

Valid credentials

Invalid credentials

Missing credentials

Logout

Session behavior

11. Test Naming Convention
    Use descriptive test names.

Good
should login successfully with valid credentials
should display validation when username is empty
should create a new employee with valid details
should display employee when searched by employee ID
should approve a pending leave request

Avoid
test1
login test
employee test
check page

12. Test Tags
    Use tags when useful for test organization.

Examples:

test('@smoke user should login successfully', async ({ page }) => {
});

test('@regression should create employee', async ({ page }) => {
});

Suggested tags:

@smoke

@regression

@login

@admin

@pim

@leave

@attendance

@recruitment

13. Waiting Strategy
    Do not use unnecessary hard waits.

Avoid
await page.waitForTimeout(5000);

Prefer Playwright's automatic waiting and assertions:

await expect(page.getByText('Dashboard')).toBeVisible();

If waiting for a specific API or navigation event is necessary, use Playwright's built-in mechanisms.

14. Handling Dynamic Data
    Employee names, IDs, usernames, and other unique values should be generated when necessary.

Example:

const employeeName = `Test Employee ${Date.now()}`;

Prefer deterministic test data when possible.

If a test creates data, consider cleaning it up after the test.

15. Negative Testing
    Every major form should have negative test cases.

Examples:

Required field validation

Invalid input

Duplicate data

Invalid date range

Unauthorized access

Invalid credentials

Unsupported file upload

Maximum field length

Special characters

Invalid email format

16. Example Test
    import { test, expect } from '@playwright/test';
    import { LoginPage } from '../../pages/LoginPage';

test.describe('OrangeHRM Login', () => {

test('should login successfully with valid credentials', async ({ page }) => {
const loginPage = new LoginPage(page);

    await page.goto('/');

    await loginPage.login(
      process.env.ORANGEHRM_USERNAME!,
      process.env.ORANGEHRM_PASSWORD!
    );

    await expect(page).toHaveURL(/dashboard/);
    await expect(
      page.getByText('Dashboard')
    ).toBeVisible();

});

});

17. Test Independence
    Tests should not depend on another test passing.

Avoid:

Test 1 creates employee
Test 2 edits employee created by Test 1
Test 3 deletes employee created by Test 1

Instead, each test should create or locate the required state independently whenever practical.

18. Error Handling
    Do not hide failures with broad try/catch blocks.

Avoid:

try {
await page.getByRole('button').click();
} catch {
// ignore
}

A failed action should normally fail the test.

Use assertions and Playwright's built-in reporting to identify failures.

19. Maintainability Rules
    When creating or modifying automation:

Prefer reusable page methods.

Avoid duplicated locators.

Avoid hard-coded waits.

Avoid brittle selectors.

Keep tests focused on business behavior.

Keep test data separate from page objects.

Keep credentials outside source code.

Use TypeScript types.

Keep methods small and descriptive.

Do not duplicate navigation logic unnecessarily.

Refactor repeated workflows into reusable functions or fixtures.

20. Definition of Done
    A test is considered complete when:

The test has a clear business scenario.

Appropriate page objects are implemented.

Stable locators are used.

Positive and relevant negative cases are covered.

Assertions verify the expected result.

The test can run independently.

No unnecessary hard waits are present.

Test data is handled safely.

The test passes consistently across supported browsers.

Failure artifacts are available through Playwright reporting.

21. AI Coding Instructions
    When generating or modifying Playwright automation code for this project:

Inspect existing page objects before creating new ones.

Reuse existing methods and fixtures whenever possible.

Do not duplicate locators or workflows.

Follow the existing project naming conventions.

Use TypeScript.

Use Playwright Test APIs.

Prefer semantic locators.

Do not use waitForTimeout() unless there is a documented reason.

Add assertions for business outcomes.

Keep tests independent.

Do not hard-code credentials.

Do not expose secrets in logs or source code.

Update the relevant page object when a locator/action belongs to that page.

Keep test files focused on scenarios rather than implementation details.

When fixing a failing test, identify whether the issue is with the locator, synchronization, test data, application behavior, or test implementation before changing the test.

Do not weaken assertions merely to make a test pass.

Preserve existing working tests when adding new functionality.

Run the relevant Playwright tests after making changes.

22. Primary Automation Goal
    The automation suite should provide reliable end-to-end coverage of critical OrangeHRM business workflows while remaining easy to maintain as the application UI changes.

Priority should be given to:

Critical business workflows

Stable and maintainable locators

Reliable synchronization

Meaningful assertions

Reusable page objects and fixtures

Independent and repeatable tests

Clear Playwright reports

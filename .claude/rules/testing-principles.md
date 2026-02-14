---
description: General testing philosophy — what to test, how to name tests, and common pitfalls
globs: ["src/**", "tests/**", "**/*.test.*", "**/*.spec.*"]
---

# Testing Principles

## Test Behavior, Not Implementation

Tests should verify what the code does, not how it does it internally. If you refactor the internals and the behavior stays the same, tests should still pass.

```typescript
// BAD: Testing implementation details
it('calls setState with the new value', () => {
  const setSpy = vi.spyOn(component, 'setState')
  component.updateQuantity(5)
  expect(setSpy).toHaveBeenCalledWith({ quantity: 5 })
})

// BAD: Testing internal method calls
it('calls validateEmail before submitting', () => {
  const spy = vi.spyOn(service, 'validateEmail')
  service.submitForm(data)
  expect(spy).toHaveBeenCalled()
})

// GOOD: Testing the observable outcome
it('updates the displayed quantity when changed', () => {
  render(<QuantitySelector />)
  fireEvent.change(screen.getByRole('spinbutton'), { target: { value: '5' } })
  expect(screen.getByRole('spinbutton')).toHaveValue(5)
})

// GOOD: Testing the result, not the steps
it('rejects submission with an invalid email', async () => {
  const result = await service.submitForm({ email: 'not-an-email' })
  expect(result.success).toBe(false)
  expect(result.errors).toContain('Invalid email address')
})
```

## Test Names Describe Scenario and Outcome

A test name should tell you exactly what's being tested without reading the code.

```typescript
// BAD: Vague names
it('works correctly')
it('handles edge case')
it('should submit')
it('test validation')

// GOOD: Scenario → outcome
it('returns null when the user ID does not exist')
it('disables the submit button while the form is submitting')
it('shows an error message when the API returns 500')
it('sends a notification when the event trigger fires')
it('redirects to login when the session token is expired')
```

## Arrange, Act, Assert

Every test follows this structure. Separate the three phases clearly.

```typescript
it('doubles the rate limit for enterprise users', () => {
  // Arrange
  const plan = createPlan({ endpoints: [{ path: '/api/data', limit: 100 }] })
  const user = createUser({ tier: 'enterprise' })

  // Act
  const limit = calculateRateLimit({ plan, user })

  // Assert
  expect(limit).toBe(200)
})
```

## Don't Test Private Functions

If a function isn't exported, it shouldn't have its own test. Test it through the public API that uses it.

```typescript
// BAD: Testing an internal helper directly
import { normalizeTimezone } from './scheduler-internals'

it('returns -8 for Pacific time', () => {
  expect(normalizeTimezone('America/Los_Angeles')).toBe(-8)
})

// GOOD: Test through the public function
import { getNextRunTime } from './scheduler'

it('schedules the next run in Pacific time', () => {
  const nextRun = getNextRunTime({
    cronExpression: '0 9 * * *',
    timezone: 'America/Los_Angeles',
  })
  expect(nextRun.getUTCHours()).toBe(17)
})
```

## Don't Mock What You Don't Own

Don't mock third-party libraries directly. Wrap them in your own interface, then mock the wrapper.

```typescript
// BAD: Mocking a library directly — breaks when the library changes
vi.mock('nodemailer', () => ({
  createTransport: vi.fn(() => ({
    sendMail: vi.fn(),
  })),
}))

// GOOD: Wrap the library, mock the wrapper
// src/lib/email/send-email.ts
export const sendEmail = async ({ to, subject, body }: EmailParams) => {
  return transporter.sendMail({ to, subject, html: body })
}

// test
vi.mock('@/lib/email/send-email')
const mockSendEmail = vi.mocked(sendEmail)
mockSendEmail.mockResolvedValue({ messageId: 'msg_123', accepted: ['user@test.com'] })
```

## Use Factories for Test Data

Don't repeat object literals across tests. Create factory functions that produce valid defaults with overrides.

```typescript
// BAD: Duplicated test data everywhere
it('test 1', () => {
  const user = { id: '1', name: 'John', email: 'john@test.com', role: 'admin' }
})
it('test 2', () => {
  const user = { id: '2', name: 'Jane', email: 'jane@test.com', role: 'user' }
})

// GOOD: Factory with sensible defaults
const createUser = (overrides?: Partial<User>): User => ({
  id: '1',
  name: 'Test User',
  email: 'test@test.com',
  role: 'user',
  ...overrides,
})

it('grants admin access to admin users', () => {
  const admin = createUser({ role: 'admin' })
  expect(canAccessAdminPanel({ user: admin })).toBe(true)
})

it('denies admin access to regular users', () => {
  const user = createUser({ role: 'user' })
  expect(canAccessAdminPanel({ user })).toBe(false)
})
```

## No Tests That Test the Framework

Don't test that React renders, that useState works, or that fetch returns a promise. Trust the framework — test your logic.

```typescript
// BAD: Testing that React renders at all
it('renders without crashing', () => {
  render(<StatusBadge status="active" />)
})

// BAD: Testing that useState updates
it('updates state when setState is called', () => {
  const [value, setValue] = renderHook(() => useState(0))
  act(() => setValue(1))
  expect(value).toBe(1)
})

// GOOD: Test YOUR component's behavior
it('shows the warning icon when status is degraded', () => {
  render(<StatusBadge status="degraded" message="High latency detected" />)
  expect(screen.getByText('High latency detected')).toBeInTheDocument()
  expect(screen.getByRole('img', { name: 'warning' })).toBeInTheDocument()
})
```

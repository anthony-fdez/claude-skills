---
description: Error handling patterns — throw at the source, handle at the boundary, never swallow
globs: ["src/**"]
---

# Error Handling

## Never Swallow Errors

Every `catch` must either re-throw, log, or handle the error meaningfully.

```typescript
// BAD: Silent swallow
try {
  await saveDocument({ doc });
} catch {
  // nothing
}

// BAD: Catching just to return null with no logging
try {
  return await fetchUser({ id });
} catch {
  return null;
}

// GOOD: Log and return null
try {
  return await fetchUser({ id });
} catch (error) {
  logger.error("[User] Failed to fetch", { error, metadata: { userId: id } });
  return null;
}

// GOOD: Re-throw if you can't handle it here
try {
  return await fetchUser({ id });
} catch (error) {
  throw new UserFetchError({ userId: id, cause: error });
}
```

## Throw at the Source, Handle at the Boundary

Low-level code should throw. High-level code (API routes, page components, event handlers) should catch and handle.

```typescript
// LOW LEVEL: Just throw
const parseConfig = ({ raw }: { raw: string }) => {
  const parsed = ConfigSchema.safeParse(JSON.parse(raw));
  if (!parsed.success) {
    throw new ConfigError({ issues: parsed.error.issues });
  }
  return parsed.data;
};

// BOUNDARY: API route catches and responds
export async function POST(request: NextRequest) {
  try {
    const config = parseConfig({ raw: await request.text() });
    return Response.json(config);
  } catch (error) {
    logger.error("[Config] Parse failed", { error });
    return createErrorResponse({
      code: "INVALID_CONFIG",
      message: "Configuration is invalid.",
      status: 400,
    });
  }
}
```

## API Routes: Log Full, Return Safe

Server-side logs get the full error. Client-side responses get a sanitized message.

```typescript
// BAD: Leaking internals to the client
return res.status(500).json({
  error: error.message, // Could contain DB queries, file paths, etc.
  stack: error.stack,
});

// GOOD: Full detail in logs, safe message to client
logger.error("[Upload] File processing failed", {
  error,
  metadata: { fileId, userId, durationMs },
});

return createErrorResponse({
  code: "UPLOAD_FAILED",
  message: "Unable to process your file. Please try again.",
  status: 500,
});
```

## No try/catch Around Code That Can't Throw

Don't wrap synchronous, infallible code in try/catch.

```typescript
// BAD: None of this can throw
try {
  const total = entries.reduce((sum, entry) => sum + entry.duration, 0);
  const average = total / entries.length;
  return { total, average };
} catch (error) {
  return { total: 0, average: 0 };
}

// GOOD: Just write the code
const total = entries.reduce((sum, entry) => sum + entry.duration, 0);
const average = total / entries.length;
return { total, average };
```

## Error Messages Say What Failed AND What to Do

```typescript
// BAD: Useless messages
throw new Error("Invalid input");
throw new Error("Something went wrong");
throw new Error("Error");

// GOOD: What failed + what to do about it
throw new Error("DATADOG_TOKEN is not set — add it to .env.local");
throw new Error("Template ID is required — pass { templateId } to getTemplate()");
throw new Error(
  "Upload rejected — the file exceeds the 10MB size limit",
);
```

## Use Error Subclasses for Different Handling

```typescript
// Define specific errors for specific handling
class ValidationError extends Error {
  constructor(public issues: z.ZodIssue[]) {
    super("Validation failed");
    this.name = "ValidationError";
  }
}

class NotFoundError extends Error {
  constructor({ resource, id }: { resource: string; id: string }) {
    super(`${resource} with id ${id} not found`);
    this.name = "NotFoundError";
  }
}

// Handle differently at the boundary
if (error instanceof ValidationError) {
  return createErrorResponse({
    code: "VALIDATION_ERROR",
    message: "Invalid request",
    status: 400,
  });
}
if (error instanceof NotFoundError) {
  return createErrorResponse({
    code: "NOT_FOUND",
    message: error.message,
    status: 404,
  });
}
```

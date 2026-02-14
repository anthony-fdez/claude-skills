---
description: Import ordering, path aliases, and module conventions
globs: ["src/**"]
---

# Import Conventions

## Use `import type` for Type-Only Imports

```typescript
// BAD: Regular import for types
import { User, Project } from "./types";

// GOOD: Explicit type import
import type { User, Project } from "./types";

// GOOD: Inline type import when mixing
import { formatDate, type DateConfig } from "./dates";
```

## Absolute Imports via `@/` Path Alias

Never use relative paths beyond `./` (same directory). Anything outside the current directory uses `@/`.

```typescript
// BAD: Relative path traversal
import { Button } from "../../components/_ui/Button";
import type { Project } from "../../../types/project.types";

// GOOD: Absolute with alias
import { Button } from "@/components/_ui/Button";
import type { Project } from "@/types/project.types";

// GOOD: Same-directory relative is fine
import { helperFn } from "./helper";
import type { LocalType } from "./types";
```

## Import Group Order

Separate groups with a blank line. Order:

1. External libraries
2. Internal absolute (`@/`)
3. Relative (`./`)
4. Type-only imports (if not inlined)

```typescript
import { useState } from "react";
import { useQuery } from "@tanstack/react-query";
import { z } from "zod";

import { Button } from "@/components/_ui/Button";
import { logger } from "@/lib/logger";
import { useGlobalStore } from "@/store/useGlobalStore";

import { buildQuery } from "./build-query";

import type { FilterOption } from "@/types/filter.types";
```

## No Barrel Files

Don't create `index.ts` files that just re-export from other files. Import directly from the source.

```typescript
// BAD: index.ts barrel
// components/index.ts
export { Button } from "./Button";
export { Input } from "./Input";
export { Dialog } from "./Dialog";

// BAD: Importing from barrel
import { Button, Input, Dialog } from "@/components";

// GOOD: Import from source
import { Button } from "@/components/_ui/Button";
import { Input } from "@/components/_ui/Input";
import { Dialog } from "@/components/_ui/Dialog";
```

## No Unused Imports

Every import must be used. Don't leave imports "for later."

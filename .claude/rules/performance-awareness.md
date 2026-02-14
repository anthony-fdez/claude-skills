---
description: Common performance pitfalls to avoid — imports, memoization, re-renders, images
globs: ["src/**"]
---

# Performance Awareness

## Import Only What You Need

Don't import entire libraries when you need one function. Tree-shaking doesn't always save you.

```typescript
// BAD: Imports the entire library
import _ from "lodash";
const sorted = _.sortBy(items, "name");

import * as dateFns from "date-fns";
const formatted = dateFns.format(date, "yyyy-MM-dd");

// GOOD: Import specific functions
import sortBy from "lodash/sortBy";
const sorted = sortBy(items, "name");

import { format } from "date-fns";
const formatted = format(date, "yyyy-MM-dd");
```

## Memoize Expensive Computations, Not Cheap Ones

`useMemo` and `useCallback` have overhead. Only use them when the computation is actually expensive or when referential stability matters.

```typescript
// BAD: Memoizing trivial operations
const fullName = useMemo(
  () => `${user.firstName} ${user.lastName}`,
  [user.firstName, user.lastName],
);

const isActive = useMemo(() => user.status === "active", [user.status]);

// GOOD: Just compute it — this is instant
const fullName = `${user.firstName} ${user.lastName}`;
const isActive = user.status === "active";

// GOOD: Memoize when it's actually expensive
const sortedEntries = useMemo(
  () => entries.toSorted((a, b) => a.title.localeCompare(b.title)),
  [entries],
);

const filteredResults = useMemo(
  () => items.filter((item) => matchesSearchQuery({ item, query })),
  [items, query],
);

// GOOD: useCallback when passed as a prop to memoized children
const handleUpdate = useCallback(
  ({ id }: { id: string }) => {
    updateItem({ id });
  },
  [updateItem],
);
```

## Stable References to Prevent Re-Renders

Objects and arrays created inline during render get a new reference every time, triggering re-renders in children.

```typescript
// BAD: New object reference every render
<UserProfile style={{ marginTop: 10 }} />
<FileList filters={{ type: 'image' }} />
<Chart data={items.map((i) => i.value)} />

// GOOD: Stable references
const profileStyle = { marginTop: 10 } // outside component or useMemo

const filters = useMemo(() => ({ type: 'image' }), [])
<FileList filters={filters} />

const chartData = useMemo(() => items.map((i) => i.value), [items])
<Chart data={chartData} />
```

## Avoid N+1 Queries

Don't make one API call per item in a list. Batch them.

```typescript
// BAD: One request per item
const enrichedUsers = await Promise.all(
  users.map(async (user) => {
    const avatar = await fetchAvatar({ userId: user.id });
    return { ...user, avatar };
  }),
);

// GOOD: Batch request
const userIds = users.map((u) => u.id);
const avatarMap = await fetchAvatarsBatch({ userIds });
const enrichedUsers = users.map((user) => ({
  ...user,
  avatar: avatarMap[user.id],
}));
```

## Images Always Have Dimensions

Images without width/height cause layout shift (CLS). Always specify dimensions and use `next/image`.

```tsx
// BAD: No dimensions, no optimization
<img src={user.avatar} alt={user.name} />

// BAD: next/image without dimensions
<Image src={user.avatar} alt={user.name} />

// GOOD: Proper next/image usage
<Image
  src={user.avatar}
  alt={user.name}
  width={400}
  height={300}
  priority={isAboveFold}
/>
```

## Lazy Load Heavy Components

Don't ship large components in the initial bundle if they're not immediately visible.

```tsx
import dynamic from "next/dynamic";

// GOOD: Heavy component loaded on demand
const RichTextEditor = dynamic(() => import("@/components/RichTextEditor"), {
  loading: () => <Skeleton className="h-64" />,
  ssr: false,
});

const InteractiveChart = dynamic(
  () => import("@/components/InteractiveChart"),
  {
    loading: () => <Skeleton className="h-96" />,
  },
);
```

## Don't Block Rendering with Synchronous Work

Heavy synchronous operations block the main thread. Defer or break them up.

```typescript
// BAD: Parsing a huge JSON blob synchronously in a component
const HugeList = ({ rawData }: { rawData: string }) => {
  const parsed = JSON.parse(rawData) // blocks rendering
  return <List items={parsed} />
}

// GOOD: Parse in an effect or during data fetching, before it reaches the component
const HugeList = ({ items }: { items: Item[] }) => {
  return <List items={items} />
}
```

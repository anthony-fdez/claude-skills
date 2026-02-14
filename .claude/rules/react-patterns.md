---
description: Universal React rules — state, effects, component structure, and common anti-patterns
globs: ["src/**/*.tsx", "src/**/*.jsx"]
---

# React Patterns

## Component Order

Every component follows this order. No exceptions.

```tsx
const NotificationCard = ({ notification, onDismiss }: NotificationCardProps) => {
  // 1. Hooks (useState, useQuery, useTranslate, etc.)
  const translate = useTranslate();
  const [isExpanded, setIsExpanded] = useState(false);
  const { data: sender } = useGetUserQuery({ id: notification.senderId });

  // 2. Derived values (computed from props/state/query data)
  const isUnread = notification.readAt === null;
  const timeAgo = formatRelativeTime({ date: notification.createdAt });

  // 3. Event handlers
  const handleDismiss = () => {
    onDismiss({ notificationId: notification.id });
  };

  // 4. Render
  return (
    <div>
      <h3>{sender?.name}</h3>
      <span>{timeAgo}</span>
      <Button onClick={handleDismiss} variant={isUnread ? "primary" : "ghost"}>
        {translate("dismiss")}
      </Button>
    </div>
  );
};
```

## No useEffect for Derived State

If a value can be computed from props, state, or query data, compute it during render.

```tsx
// BAD: useEffect + useState to derive a value
const [fullName, setFullName] = useState("");
useEffect(() => {
  setFullName(`${user.firstName} ${user.lastName}`);
}, [user.firstName, user.lastName]);

// GOOD: Compute during render
const fullName = `${user.firstName} ${user.lastName}`;

// GOOD: useMemo for expensive computations
const sortedMessages = useMemo(
  () => messages.toSorted((a, b) => a.sentAt - b.sentAt),
  [messages],
);
```

## No useEffect for Event Responses

If something should happen when an event occurs, put the logic in the event handler.

```tsx
// BAD: useEffect watching state to trigger side effect
const [isPanelOpen, setIsPanelOpen] = useState(false);

useEffect(() => {
  if (isPanelOpen) {
    analyticsService.track(AnalyticsEvent.OPEN_PANEL, { panelId });
  }
}, [isPanelOpen]);

// GOOD: Track in the handler that opens the panel
const handleOpenPanel = () => {
  setIsPanelOpen(true);
  analyticsService.track(AnalyticsEvent.OPEN_PANEL, { panelId });
};
```

## Props: Callbacks Use `onX`, Never Pass Setters

```tsx
// BAD: Exposing state setter
<VolumeSlider
  level={volume}
  setLevel={setVolume}
/>

// GOOD: Named callback with parent control
<VolumeSlider
  level={volume}
  onLevelChange={handleVolumeChange}
/>

// Parent controls the logic
const handleVolumeChange = ({ level }: { level: number }) => {
  if (level < 0 || level > MAX_VOLUME) return
  setVolume(level)
}
```

## No Inline Complex Logic in JSX

Extract complex conditionals and handlers out of JSX.

```tsx
// BAD: Logic buried in JSX
{
  !isFocusMode && !isLoading && !isCompact && willShowSuggestions && (
    <RelatedContent contentId={contentId} />
  );
}

// GOOD: Named boolean
const shouldShowSuggestions =
  !isFocusMode && !isLoading && !isCompact && willShowSuggestions;

{
  shouldShowSuggestions && <RelatedContent contentId={contentId} />;
}
```

```tsx
// BAD: 20+ line onClick handler inline
<Button onClick={() => {
  if (condition) { ... }
  trackEvent(...)
  updateState(...)
  navigate(...)
}}>

// GOOD: Extract to named handler
const handleSubmit = () => {
  if (condition) { ... }
  trackEvent(...)
  updateState(...)
  navigate(...)
}

<Button onClick={handleSubmit}>
```

## Extract Components, Not Render Functions

```tsx
// BAD: Render function inside component (recreated every render)
const DashboardPage = () => {
  const renderSidebar = () => {
    return <div>...</div>;
  };

  return <main>{renderSidebar()}</main>;
};

// GOOD: Separate component
const DashboardSidebar = ({ stats }: DashboardSidebarProps) => {
  return <div>...</div>;
};

const DashboardPage = () => {
  return (
    <main>
      <DashboardSidebar stats={stats} />
    </main>
  );
};
```

## API Data Stays in React Query

Never sync React Query data into Zustand or useState. React Query is the single source of truth for server state.

```tsx
// BAD: Syncing query data to global store
const { data: teammates } = useGetTeammatesQuery();

useEffect(() => {
  if (teammates) {
    setProject({ lead: teammates.find((t) => t.isLead) });
  }
}, [teammates]);

// GOOD: Derive from query data directly
const { data: teammates } = useGetTeammatesQuery();
const projectLead = teammates?.find((t) => t.isLead) ?? null;

return <TeamList teammates={teammates} lead={projectLead} />;
```

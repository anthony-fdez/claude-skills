---
name: writing-react
description: Enforces React component patterns, state management, and anti-patterns. Use when writing any React component, handling state, or using hooks.
---

# Writing React Components

## Component Structure

```tsx
const ProductCard = ({ product, onAddToCart }: ProductCardProps) => {
  // 1. Hooks first
  const [isExpanded, setIsExpanded] = useState(false)
  const { data: reviews } = useGetReviewsQuery({ productId: product.id })

  // 2. Event handlers
  const handleAddToCart = () => {
    onAddToCart(product.id)
  }

  // 3. Render
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <Button onClick={handleAddToCart}>Add to Cart</Button>
    </div>
  )
}
```

Arrow function components only. No `function` declarations.

## Props: Pass Callbacks, Not Setters

```tsx
// ❌ Bad
<QuantityControls itemCount={itemCount} setItemCount={setItemCount} />

// ✅ Good
<QuantityControls itemCount={itemCount} onItemCountChange={handleItemCountChange} />
```

## State Anti-Patterns

### Don't Sync Props to State

```tsx
// ❌ Bad: useState + useEffect to sync
const [selectedItem, setSelectedItem] = useState(getDefault(items, query))
useEffect(() => { setSelectedItem(getDefault(items, query)) }, [query.id])

// ✅ Good: Derive during render
const selectedItem = useMemo(
  () => getDefault(items, query),
  [items, query.id],
)
```

### Don't Store Computed Values

```tsx
// ❌ Bad
const [hasMultipleItems, setHasMultipleItems] = useState(false)
useEffect(() => { setHasMultipleItems(items?.length > 1) }, [items])

// ✅ Good
const hasMultipleItems = (items?.length ?? 0) > 1
```

## useEffect Anti-Patterns

### No useEffect for Events — Track in the Handler

```tsx
// ❌ Bad
useEffect(() => { if (isModalOpen) analytics.track('modal_viewed') }, [isModalOpen])

// ✅ Good
const handleOpenModal = () => {
  setUI({ isModalOpen: true })
  analytics.track('modal_viewed')
}
```

### No useEffect for Data Fetching — Use React Query

```tsx
// ❌ Bad
const [data, setData] = useState(null)
useEffect(() => { fetchData().then(setData) }, [deps])

// ✅ Good
const { data, isLoading } = useQuery({ queryKey: ['data', deps], queryFn: fetchData })
```

## JSX Patterns

### Extract Complex Conditionals

```tsx
// ❌ Bad: Logic buried in JSX
{!isFocusMode && !isLoading && !isCompact && willShowSuggestions && <RelatedContent />}

// ✅ Good: Named boolean
const shouldShowSuggestions = !isFocusMode && !isLoading && !isCompact && willShowSuggestions
{shouldShowSuggestions && <RelatedContent />}
```

### Extract Components, Not Render Functions

```tsx
// ❌ Bad: Render function recreated every render
const renderDescription = () => { ... }
return <div>{renderDescription()}</div>

// ✅ Good: Separate component
<ProductDescription isCompact={isCompact} item={selectedItem} />
```

### No Inline Complex Logic

Extract handlers longer than a few lines into named functions.

## Context: Don't Overuse

```tsx
// ❌ Bad: Context with frequently changing values — use Zustand instead
// ❌ Bad: Context for 2-level prop passing — just pass props

// ✅ Good: Zustand for frequently changing state (components subscribe to only what they need)
// ✅ Good: Composition pattern to avoid prop drilling
```

## Data Flow: Single Source of Truth

API data stays in React Query — never sync to Zustand or useState.

```tsx
const { data: items } = useGetItemsQuery()
const defaultItem = items?.find((item) => item.isDefault) ?? null
return <ItemSelector items={items} default={defaultItem} />
```

## Checklist

- [ ] Hooks → handlers → render order
- [ ] Props use `onX` naming for callbacks (not setters)
- [ ] Derived state computed, not stored in useState
- [ ] No useEffect for event handling or data fetching
- [ ] Complex conditionals extracted to named booleans
- [ ] Components extracted (not render functions)
- [ ] API data stays in React Query only
- [ ] `data-test` attributes added for E2E testing

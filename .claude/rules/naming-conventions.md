---
description: Naming patterns for variables, functions, components, and types
globs: ["src/**"]
---

# Naming Conventions

## Variables Describe What They Hold

Name variables after what they represent, not how they were obtained or what type they are.

```typescript
// BAD: Implementation details in the name
const fetchedUserData = await getUser({ id })
const apiResponseBody = await response.json()
const filteredItemsList = items.filter((i) => i.active)

// GOOD: What it actually is
const user = await getUser({ id })
const project = await response.json()
const activeItems = items.filter((i) => i.active)
```

## No Type Information in Names

The type system handles this. Don't encode types into names.

```typescript
// BAD: Type in the name
const userArray = [user1, user2]
const countNumber = items.length
const nameString = user.name
const isActiveBoolean = user.active
const userObject = { name: 'John', age: 30 }
const onClickFunction = () => {}

// GOOD: Just the concept
const users = [user1, user2]
const count = items.length
const name = user.name
const isActive = user.active
const user = { name: 'John', age: 30 }
const onClick = () => {}
```

## Booleans Use Prefixes

Booleans always start with `is`, `has`, `should`, `can`, or `was`.

```typescript
// BAD: Ambiguous — is this a boolean or a string?
const active = user.active
const items = playlist.tracks.length > 0
const admin = user.role === 'admin'

// GOOD: Clearly a boolean
const isActive = user.active
const hasTracks = playlist.tracks.length > 0
const isAdmin = user.role === 'admin'
const canEdit = user.permissions.includes('edit')
const shouldRedirect = !isAuthenticated
const wasDeleted = response.status === 204
```

## Event Handlers Use `handle` + Noun + Verb

```typescript
// BAD: Vague or inconsistent naming
const click = () => {}
const onClickHandler = () => {}
const doSubmit = () => {}
const profileUpdate = () => {}

// GOOD: handle + what + action
const handleProfileUpdate = () => {}
const handleFormSubmit = () => {}
const handleModalClose = () => {}
const handleSearchClear = () => {}

// GOOD: Short form when context is obvious (e.g., inside a <SearchBar>)
const handleClear = () => {}
const handleSubmit = () => {}
```

## Props Callbacks Use `on` + Noun + Verb

```typescript
// BAD
<Modal close={handleClose} />
<Form submitForm={handleSubmit} />

// GOOD: on + what happens
<Modal onClose={handleClose} />
<Form onSubmit={handleSubmit} />
<SearchBar onSearch={handleSearch} onClear={handleClear} />
```

## No Abbreviations (Except Universal Ones)

Spell things out. The only acceptable abbreviations are ones any developer would recognize instantly.

```typescript
// Allowed abbreviations
id, url, ref, props, params, config, auth, env, db, api, src, btn, img, nav, max, min

// BAD: Unclear abbreviations
const usr = getUser()
const proj = getProject()
const qty = item.quantity
const addr = user.address
const mgr = new Manager()
const tmpl = loadTemplate()
const cb = () => {}
const fn = () => {}

// GOOD: Spelled out
const user = getUser()
const project = getProject()
const quantity = item.quantity
const address = user.address
const manager = new Manager()
const template = loadTemplate()
```

## Collections Are Plural, Items Are Singular

```typescript
// BAD: Unclear what's a list vs a single item
const project = await fetchProjects() // returns array
const member = team.members // this is the array

// GOOD: Plural for collections, singular for items
const projects = await fetchProjects()
const teamMembers = team.members
const firstMember = teamMembers[0]

projects.map((project) => project.name)
teamMembers.forEach((member) => notifyMember({ member }))
```

## Constants Are UPPER_SNAKE_CASE

Only for true constants — values known at compile time that never change.

```typescript
// GOOD: Compile-time constants
const MAX_RETRY_ATTEMPTS = 3
const API_BASE_URL = 'https://api.example.com'
const DEFAULT_PAGE_SIZE = 20

const ROUTE_SEGMENTS = {
  COUNTRY_CODE: 'c',
  EVENT: 'e',
} as const

// BAD: Using UPPER_CASE for runtime values
const USER_DATA = await fetchUser() // not a constant
const ITEMS_COUNT = items.length // computed at runtime
```

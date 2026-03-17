# React Conventions

Frontend conventions for Magnitude Instruments React applications.

## Tech Stack

- **React 18** (JavaScript, no TypeScript)
- **Vite** — build tool and dev server
- **MUI 5** — UI component library
- **React Router v6** — client-side routing
- **AWS Amplify v6** — Auth and API modules (tree-shakeable imports)

## Project Structure

```
src/
├── App.jsx                    # Root component, routing, providers
├── aws-exports.js             # AWS config (manually managed)
├── theme.js                   # MUI theme + color constants
├── index.css                  # Global styles, CSS custom properties
├── context/                   # React contexts (one per domain)
│   ├── AuthContext.jsx
│   └── FileDataContext.jsx
├── styles/                    # Shared style objects
│   └── formStyles.js
├── components/
│   ├── Authentication/        # Auth-related components
│   ├── Body/                  # Main content area
│   ├── Header/                # Navigation
│   ├── misc/                  # Shared UI components
│   └── styles/                # Component CSS files
└── img/                       # Static images
```

## Component Conventions

### Functional Components Only

Always use functional components with hooks. Never use class components.

```jsx
// Good
const MyComponent = ({ title, onAction }) => {
  const [value, setValue] = useState("");
  return <div>{title}</div>;
};

// Bad — class component
class MyComponent extends React.Component { ... }
```

### File Naming

| Type | Convention | Example |
|------|-----------|---------|
| Components | PascalCase `.jsx` | `FileExplorer.jsx`, `NavBar.jsx` |
| Contexts | PascalCase `.jsx` | `AuthContext.jsx`, `FileDataContext.jsx` |
| Utilities | camelCase `.js` | `formatAuthError.js`, `formStyles.js` |
| Styles | Component name `.css` | `FileExplorer.css`, `NavBar.css` |

### Exports

- **Components**: default export
- **Contexts**: named export for provider and hook, default export for context object
- **Utilities**: named export

```jsx
// Component
const FileExplorer = () => { ... };
export default FileExplorer;

// Context
export function AuthProvider({ children }) { ... }
export function useAuth() { ... }
export default AuthContext;

// Utility
export function formatAuthError(error) { ... }
```

### Component Size

Keep components under 200 lines. If a component exceeds this, extract sub-components:

```jsx
// Instead of one 400-line NavBar, split into:
// NavBar.jsx — layout and routing
// AccountMenu.jsx — account dropdown logic
// ScrollBehavior.jsx — scroll-based style changes (or a custom hook)
```

### Never Store JSX in State

State should hold data, not rendered output. Conditionally render based on state values:

```jsx
// Good — render based on state
const [view, setView] = useState("signIn");

return (
  <>
    {view === "signIn" && <SignIn />}
    {view === "createAccount" && <CreateAccount />}
    {view === "forgotPassword" && <ForgotPassword />}
  </>
);

// Bad — store JSX in state
const [content, setContent] = useState(<SignIn />);
return content;
```

## State Management

### Context Usage

Use React Context for state shared across multiple components. One context per domain:

- `AuthContext` — user, auth state, login/logout
- `FileDataContext` — S3 file/folder data

### Error State

Always expose error state from contexts that fetch data. Never silently catch errors:

```jsx
// Good — error state exposed to consumers
const [error, setError] = useState(null);

try {
  const data = await fetchData();
  setData(data);
} catch (err) {
  console.error(err);
  setError(err);
}

return (
  <Context.Provider value={{ data, loading, error }}>
    {children}
  </Context.Provider>
);

// Bad — error swallowed
try {
  const data = await fetchData();
  setData(data);
} catch (err) {
  console.log(err);  // user sees nothing
}
```

### Avoid Redundant State

Don't duplicate information that can be derived:

```jsx
// Good — derive from existing state
const { user } = useAuth();
const isSignedIn = user !== null;

// Bad — redundant state
const { user } = useAuth();
const [signedIn, setSignedIn] = useState(false);
```

## Hooks

### useEffect

- Always include a cleanup function for event listeners
- Always specify dependencies

```jsx
// Good — cleanup and dependencies
useEffect(() => {
  const handleScroll = () => { ... };
  window.addEventListener("scroll", handleScroll);
  return () => window.removeEventListener("scroll", handleScroll);
}, []);

// Bad — no cleanup, recreated every render
window.onscroll = function() { ... };
```

### useCallback and useMemo

Use `useCallback` for event handlers passed as props to child components. Use `useMemo` for expensive computations. Don't over-optimize — only add them when there's a measurable benefit or the function is a dependency of another hook.

```jsx
// Worth memoizing — passed to child component
const handleFileClick = useCallback((e, filePath) => {
  e.preventDefault();
  downloadFile(filePath);
}, []);

// Not worth memoizing — simple inline handler
<button onClick={() => setOpen(true)}>Open</button>
```

### Custom Hooks

Extract reusable logic into custom hooks in the `context/` directory:

```jsx
// useAuth.js
export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error("useAuth must be used within an AuthProvider");
  }
  return context;
}
```

## API Integration (Amplify v6)

### Imports

Use tree-shakeable imports — never import the entire Amplify library:

```jsx
// Good
import { fetchAuthSession } from "aws-amplify/auth";
import { get } from "aws-amplify/api";

// Bad
import Amplify from "aws-amplify";
```

### API Call Pattern

```jsx
const session = await fetchAuthSession();
const token = session.tokens?.idToken?.toString();

const { body } = await get({
  apiName: API_NAME,
  path: "/endpoint",
  options: { headers: { Authorization: token } },
}).response;

const data = await body.json();    // for JSON responses
const text = await body.text();    // for plain text responses
```

### API Name Constant

Define the API name once per file as a module-level constant:

```jsx
const API_NAME = "MagInstsSoftwareRepoAPI";
```

## Styling

### Approach (ordered by preference)

1. **CSS custom properties** (`:root` in `index.css`) for design tokens
2. **Component CSS files** for component-specific styles
3. **MUI `sx` prop** for MUI component overrides
4. **Shared style objects** (`formStyles.js`) for repeated patterns

### CSS Custom Properties

Define all design tokens in `:root`:

```css
:root {
  --color-primary: #3D7EDB;
  --color-success: #4caf50;
  --color-error: #f44336;
  --color-bg-page: #f5f5f5;
  --color-text-muted: #999999;
}
```

Reference in CSS files:

```css
.header {
  background-color: var(--color-primary);
}
```

### MUI Theme

One global `ThemeProvider` in `App.jsx`. Export `colors` from `theme.js` for use in JSX:

```jsx
import { colors } from "../../theme";

<CircularProgress sx={{ color: colors.primary }} />
```

### Avoid Inline Styles

Don't use `style={{}}` in JSX. Use CSS classes or MUI `sx` prop instead:

```jsx
// Good
<div className="header-container">

// Good (MUI components)
<Box sx={{ display: "flex", gap: 2 }}>

// Bad
<div style={{ display: "flex", backgroundColor: "#3D7EDB", padding: "16px" }}>
```

### No Magic Numbers

Use CSS custom properties or named constants instead of hardcoded pixel values:

```jsx
// Good
maxHeight: expanded ? "var(--accordion-max-height)" : "var(--accordion-collapsed)"

// Bad
maxHeight: expanded ? "520px" : "86px"
```

## Accessibility

### Keyboard Navigation

Never set `tabIndex="-1"` on interactive elements (links, buttons). This removes them from keyboard navigation:

```jsx
// Good — interactive element is keyboard-accessible
<a href="/downloads">Downloads</a>

// Bad — removed from tab order
<a href="/downloads" tabIndex={-1}>Downloads</a>
```

### ARIA Attributes

- Add `aria-label` to icon-only buttons and non-descriptive links
- Add `aria-expanded` to accordion/collapsible triggers
- Add `aria-live="polite"` to regions that update dynamically (error messages, loading states)

```jsx
<button aria-label="Account menu" onClick={toggleMenu}>
  <AccountIcon />
</button>

<button aria-expanded={isOpen} onClick={toggle}>
  {title}
</button>
```

### Form Fields

- Every input must have an associated `<label>` or `aria-label`
- Use `autocomplete` attributes on inputs (`email`, `current-password`, `new-password`)
- Wrap related fields in `<form>` elements

### Semantic HTML

Use semantic elements over generic `<div>`:

- `<nav>` for navigation
- `<main>` for primary content
- `<header>` / `<footer>` for page structure
- `<button>` for clickable actions (not `<div onClick>`)
- `<a>` for navigation links

## Error Handling

### User-Facing Errors

Always display errors to the user. Never silently catch:

```jsx
// Good — user sees the error
const [error, setError] = useState(null);

try {
  await apiCall();
} catch (err) {
  setError(formatAuthError(err));
}

{error && <Alert severity="error">{error}</Alert>}

// Bad — silent failure
try {
  await apiCall();
} catch (err) {
  console.log(err);
}
```

### Error Formatting

Use a shared utility for consistent error messages:

```jsx
import { formatAuthError } from "../misc/formatAuthError";

catch (err) {
  setErrorMsg(formatAuthError(err));
}
```

## Performance

### Lazy Loading

For multi-tab or multi-view components, consider lazy loading views that aren't immediately visible:

```jsx
const EditAccount = React.lazy(() => import("./EditAccount"));
```

### Avoid DOM Manipulation

Never use `document.getElementById`, `document.createElement`, or direct DOM manipulation in React components. Use refs instead:

```jsx
// Good
const containerRef = useRef(null);

// Bad
const el = document.getElementById("container");
```

Exception: creating temporary `<a>` elements for file downloads is acceptable when the HTML5 `download` attribute isn't sufficient.

### Event Listeners

Always attach event listeners in `useEffect` with cleanup — never in the render body:

```jsx
// Good
useEffect(() => {
  const handler = () => { ... };
  window.addEventListener("resize", handler);
  return () => window.removeEventListener("resize", handler);
}, []);

// Bad — new listener on every render
window.onresize = () => { ... };
```

## JavaScript Style

### Variables

- Always use `const` by default
- Use `let` only when reassignment is necessary
- Never use `var`

### Comparisons

- Use strict equality (`===` and `!==`)
- Use optional chaining for nullable access (`user?.attributes?.email`)
- Use nullish coalescing for defaults (`value ?? "default"`)

### Constants

Define magic strings and numbers as named constants:

```jsx
// Good
const API_NAME = "MagInstsSoftwareRepoAPI";
const PRESIGNED_URL_EXPIRY = 120;

// Bad
path: `/GetPresignedURL/${filePath}`,  // what API is this?
```

### Imports

Order imports consistently:

```jsx
// 1. React
import React, { useState, useEffect } from "react";

// 2. Third-party libraries
import { fetchAuthSession } from "aws-amplify/auth";
import CircularProgress from "@mui/material/CircularProgress";

// 3. Local contexts and hooks
import { useAuth } from "../../context/AuthContext";

// 4. Local components
import Accordion from "../misc/Accordion";

// 5. Styles
import "../styles/Component.css";
```

### Formatting

- Use double quotes for JSX attributes and imports
- Use template literals for string interpolation
- Use trailing commas in multi-line arrays and objects
- Use arrow functions for components and callbacks

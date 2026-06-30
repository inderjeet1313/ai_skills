# /add-to-toolkit - Add Components to dc-ui-toolkit

## CRITICAL: USER CONFIRMATION REQUIRED

**BEFORE adding any component to dc-ui-toolkit, you MUST:**

1. Explain what component you want to add and why
2. Show the proposed implementation
3. Ask the user explicitly:
   > "Should I proceed with adding **[ComponentName]** to dc-ui-toolkit?
   > This will create files in `src/components/dc-ui-toolkit/components/[Category]/[ComponentName]/`"
4. **WAIT for explicit approval** before creating any files
5. **NEVER** proceed without user confirmation

---

## Purpose

Add new reusable components to the dc-ui-toolkit design system that:
- Will be used in 3+ places across the app
- Align with the existing design system patterns
- Are general-purpose enough to warrant toolkit inclusion

---

## Directory Structure

```
src/components/dc-ui-toolkit/components/
├── Button/           # Button variants (Button, BackButtonPrimary, CircularPrimary, RoundIconLabel, etc.)
├── Cards/            # Card-based components (ActionCard, ActivityCard, WalletCard, InfoCard, etc.)
├── Currency/         # Currency display (CurrencyAmount)
├── DetailedContent/  # Copyable content card
├── Header/           # Header variants (Basic, DashboardHeader)
├── Icons/            # Icon components
├── Input/            # Form inputs (BasicInput, CurrencyAmountInput, AmountInput, SelectInput)
├── Layout/           # Screen-level layout components
│   ├── ScreenLayout/ # Flat-background screen wrapper
│   └── TabView/      # Tab container
├── Lists/            # List item components
│   └── StackedLists/
├── Overlays/         # Modals, sheets, dividers
├── ReviewDetails/    # Compound review screen layout
├── Section/          # Section header with count
├── Selectors/        # Selection components (PillGroup, Tabs, Toggle, ListSelector, etc.)
├── Loaders/          # Loading indicators
├── [Category]/       # New category if needed
│   └── [ComponentName]/
│       ├── [ComponentName].tsx
│       └── index.ts
└── index.ts          # Main exports
```

---

## Category Selection Guide

| Category | When to Use |
|----------|-------------|
| `Button/` | Interactive button variants |
| `Cards/` | Container components with card styling (pressable, info, activity, wallet cards) |
| `Currency/` | Currency display and formatting components |
| `DetailedContent/` | Content cards with copyable identifiers |
| `Header/` | Page/section header variants |
| `Icons/` | Icon wrappers or custom icons |
| `Input/` | Form input components |
| `Layout/` | Screen-level layout wrappers and tab containers |
| `Lists/StackedLists/` | List item components |
| `Overlays/` | Modals, bottom sheets, overlays |
| `ReviewDetails/` | Review/confirmation screen compound components |
| `Section/` | Section header components with count/badge |
| `Selectors/` | Selection UI (checkboxes, pills, tabs, toggle, list selector) |
| `Loaders/` | Loading indicators and skeletons |

---

## Toolkit Component Template

```typescript
// [ComponentName].tsx
import React from 'react';
import { StyleSheet, View, ViewStyle, Pressable } from 'react-native';

import { Text } from '../Text';  // Internal toolkit import
import {
  CustomFunctionComponent,
  commonProps,
  withTheme,
  FullTheme,
} from '@dc-ui-toolkit-theme';
import { wp, hp } from '@dc-ui-toolkit-utils';

// ============================================
// TYPES
// ============================================

export interface [ComponentName]Props extends commonProps {
  /**
   * Primary text content
   */
  title: string;

  /**
   * Secondary text content
   */
  subtitle?: string;

  /**
   * Visual variant
   * @default 'primary'
   */
  variant?: 'primary' | 'secondary' | 'tertiary';

  /**
   * Disable interaction
   * @default false
   */
  disabled?: boolean;

  /**
   * Press handler
   */
  onPress?: () => void;

  /**
   * Container style override
   */
  style?: ViewStyle;
}

// ============================================
// COMPONENT
// ============================================

const [ComponentName]Base: CustomFunctionComponent<[ComponentName]Props> = ({
  theme,
  isRtl,
  testID,
  title,
  subtitle,
  variant = 'primary',
  disabled = false,
  onPress,
  style,
}) => {
  const isPrimary = variant === 'primary';
  const isSecondary = variant === 'secondary';

  const getBackgroundColor = () => {
    if (disabled) return theme.palette['surface-secondary'];
    if (isPrimary) return theme.palette['buttons-and-selectors-brown-btn-default'];
    if (isSecondary) return theme.palette['surface-primary'];
    return 'transparent';
  };

  const getTextColor = () => {
    if (disabled) return theme.palette['text-disabled'];
    if (isPrimary) return theme.palette['buttons-and-selectors-brown-text-default'];
    return theme.palette['text-primary'];
  };

  const Container = onPress && !disabled ? Pressable : View;

  return (
    <Container
      style={[
        styles(theme).container,
        { backgroundColor: getBackgroundColor() },
        isSecondary && styles(theme).bordered,
        style,
      ]}
      onPress={onPress}
      disabled={disabled}
      testID={testID}
    >
      <Text
        typography={{
          type: 'BODY',
          size: isRtl ? 'arBody1Semibold' : 'enBody1Semibold',
        }}
        color={getTextColor()}
      >
        {title}
      </Text>

      {subtitle && (
        <Text
          typography={{
            type: 'BODY',
            size: isRtl ? 'arBody2' : 'enBody2',
          }}
          color={disabled
            ? theme.palette['text-disabled']
            : theme.palette['text-secondary']
          }
        >
          {subtitle}
        </Text>
      )}
    </Container>
  );
};

// ============================================
// STYLES
// ============================================

const styles = (theme: FullTheme) =>
  StyleSheet.create({
    container: {
      padding: theme.spaces[16],
      borderRadius: theme.border.radius[8],
      gap: theme.spaces[4],
    },
    bordered: {
      borderWidth: theme.border.widths[1],
      borderColor: theme.palette['input-field-field-stroke-default'],
    },
  });

// ============================================
// EXPORT
// ============================================

export const [ComponentName] = withTheme([ComponentName]Base, '[ComponentName]');
```

---

## Index File Pattern

```typescript
// [ComponentName]/index.ts
export { [ComponentName] } from './[ComponentName]';
export type { [ComponentName]Props } from './[ComponentName]';
```

---

## Category Index Update

After creating the component, update the category index:

```typescript
// [Category]/index.ts
export * from './[ExistingComponent1]';
export * from './[ExistingComponent2]';
export * from './[ComponentName]';  // Add new export
```

---

## Main Components Index Update

Update the main toolkit exports:

```typescript
// src/components/dc-ui-toolkit/components/index.ts
export * from './Button';
export * from './Cards';
export * from './Header';
// ... existing exports
export * from './[Category]';  // If new category
```

---

## Toolkit-Specific Patterns

### Using Internal Text Component
```typescript
// Import from relative path within toolkit
import { Text } from '../Text';

// NOT from alias
// import { Text } from '@dc-ui-toolkit';  // DON'T DO THIS
```

### Using withTheme HOC
```typescript
import {
  CustomFunctionComponent,
  commonProps,
  withTheme,
} from '@dc-ui-toolkit-theme';

// Component receives theme via props
const ComponentBase: CustomFunctionComponent<Props> = ({
  theme,  // Injected by withTheme
  isRtl,  // Injected by withTheme
  testID, // From commonProps
  // ... other props
}) => {
  // Use theme.palette, theme.spaces, etc.
};

// Wrap with HOC
export const Component = withTheme(ComponentBase, 'Component');
```

### Supporting Inverse Theme
```typescript
// Components wrapped with withTheme automatically support:
<Component useInverseTheme />

// This inverts the theme colors for components on dark backgrounds
```

### RTL Support
```typescript
// Access isRtl from props
const ComponentBase: CustomFunctionComponent<Props> = ({ isRtl, theme }) => {
  return (
    <Text
      typography={{
        type: 'BODY',
        // Use Arabic typography sizes for RTL
        size: isRtl ? 'arBody1' : 'enBody1',
      }}
    >
      {content}
    </Text>
  );
};
```

---

## Naming Conventions

| Item | Convention | Example |
|------|------------|---------|
| Component file | PascalCase | `MyComponent.tsx` |
| Component name | PascalCase | `MyComponent` |
| Props type | PascalCase + Props | `MyComponentProps` |
| Style function | camelCase | `styles` |
| Export file | `index.ts` | `index.ts` |

---

## Required Exports

Every toolkit component MUST export:
1. The component itself
2. The props interface/type

```typescript
// index.ts
export { [ComponentName] } from './[ComponentName]';
export type { [ComponentName]Props } from './[ComponentName]';
```

---

## Grouped Export Pattern

For related components (like Button variants):

```typescript
// Button/index.ts
import { Button as ButtonCmp } from './Button.Button';
import { BackButtonPrimary } from './Button.BackButtonPrimary';
import { CircularPrimary } from './Button.CircularPrimary';

export const Button = {
  Button: ButtonCmp,
  BackButtonPrimary,
  CircularPrimary,
  // ... other variants
};

// Usage: <Button.Button />, <Button.CircularPrimary />
```

---

## Pre-Submission Checklist

Before asking user for approval:
- [ ] Component is truly reusable (3+ use cases identified)
- [ ] Follows existing toolkit patterns
- [ ] Uses `withTheme` HOC
- [ ] Uses `CustomFunctionComponent` type
- [ ] Extends `commonProps` for testID
- [ ] Exports props type
- [ ] Uses internal Text import (not alias)
- [ ] Supports theme colors (light/dark)
- [ ] Considers RTL with isRtl prop
- [ ] Has JSDoc comments for props
- [ ] No external dependencies beyond toolkit utilities

---

## User Approval Template

When ready to add a component, present this to the user:

```markdown
## Proposed Toolkit Addition: [ComponentName]

**Purpose:** [Brief description of what it does]

**Use Cases:**
1. [Use case 1]
2. [Use case 2]
3. [Use case 3]

**Category:** [Button/Cards/Lists/etc.]

**Files to Create:**
- `src/components/dc-ui-toolkit/components/[Category]/[ComponentName]/[ComponentName].tsx`
- `src/components/dc-ui-toolkit/components/[Category]/[ComponentName]/index.ts`

**Files to Update:**
- `src/components/dc-ui-toolkit/components/[Category]/index.ts`

**Should I proceed with adding [ComponentName] to dc-ui-toolkit?**
```

---

## After User Approval

1. Create the component file
2. Create the index file
3. Update category index
4. Update main components index (if new category)
5. Verify exports work correctly

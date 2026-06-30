# /build-component - Create Reusable Components

## Purpose
Create a screen-scoped or feature-shared component following Digital Dirham Wallet standards. Before proceeding, confirm no equivalent exists in `dc-ui-toolkit`.

---

## Decision Tree

```
Is this component already in dc-ui-toolkit?
├── YES  →  Use it. Do NOT recreate.
└── NO   →  Will it be used in 3+ places across the app?
    ├── YES  →  Ask user: "Add to dc-ui-toolkit?" Use /add-to-toolkit if yes.
    └── NO   →  Is it a modified variant of an existing toolkit component?
        ├── YES  →  Ask user: "Extend toolkit component?" Use /add-to-toolkit if yes.
        └── NO   →  Create as screen/feature-specific component (proceed below).
```

### Commonly Missed Toolkit Components — Check Before Creating

Before creating a new component, verify none of these fit:

| If you need... | Use instead |
|---|---|
| Pressable card with icon + title + subtitle + arrow | `Cards.ActionCard` |
| Transaction / activity list row | `Cards.ActivityCard` |
| Visual wallet card with background image | `Cards.WalletCard` |
| Amount input with dynamic font scaling | `Input.CurrencyAmountInput` |
| Dropdown / picker trigger field | `Input.SelectInput` |
| Screen wrapper (flat background, no gradient) | `ScreenLayout` |
| Tab container with animated underline | `TabView` |
| Section header with count badge | `Section` |
| Formatted currency display | `CurrencyAmount` |
| Review/confirmation page layout | `ReviewDetails` |
| Round icon button with label below | `Button.RoundIconLabel` |
| Card with copyable wallet ID + icon | `DetailedContent` |
| Animated on/off switch | `Selectors.Toggle` |
| Simple list item with icon + text | `Selectors.ListSelector` |

---

## File Locations

| Scope | Location |
|-------|----------|
| 1 screen only | `src/scenes/[FeatureName]/components/[ComponentName].tsx` |
| 2–3 screens / same feature | `src/components/base-components/[ComponentName]/` |
| App-wide (ask user first) | `src/components/dc-ui-toolkit/components/[ComponentName]/` |

Always create an `index.ts` barrel export alongside the component.

---

## Template 1: Standard Component (useTheme hook pattern)

Use this for most components.

```typescript
// [ComponentName].tsx
import React, { useCallback, useMemo } from 'react';
import { I18nManager, StyleSheet, View, Pressable } from 'react-native';
import type { ViewStyle } from 'react-native';

import { Icon, Text as UIText } from '@dc-ui-toolkit';
import { FullTheme, useTheme } from '@dc-ui-toolkit-theme';
import { useThemedStyles } from '@hooks/useThemedStyles';
import { useTranslation } from '@hooks/useTranslation';
import { wp } from '@dc-ui-toolkit-utils';

// ============================================================
// TYPES
// ============================================================

export interface [ComponentName]Props {
  /** Required content */
  title: string;

  /** Optional content */
  subtitle?: string;
  iconName?: string;

  /** Interaction */
  onPress?: () => void;

  /** State */
  disabled?: boolean;
  isLoading?: boolean;

  /** Style override (always allow — Open/Closed principle) */
  style?: ViewStyle;
  contentStyle?: ViewStyle;

  /** Testing */
  testID?: string;
}

// ============================================================
// COMPONENT
// ============================================================

export const [ComponentName]: React.FC<[ComponentName]Props> = ({
  title,
  subtitle,
  iconName,
  onPress,
  disabled = false,
  isLoading = false,
  style,
  contentStyle,
  testID,
}) => {
  const { theme } = useTheme();
  const styles = useThemedStyles(createStyles);
  const { isRTL } = useTranslation();
  const isRtlLayout = I18nManager.isRTL;

  // Derived
  const isInteractive = Boolean(onPress) && !disabled;
  const Container = isInteractive ? Pressable : View;

  const containerProps = isInteractive
    ? { onPress, disabled, testID }
    : { testID };

  return (
    <Container
      style={[
        styles.container,
        isRtlLayout && styles.containerRTL,
        disabled && styles.containerDisabled,
        style,
      ]}
      {...containerProps}
    >
      {iconName && (
        <View style={styles.iconWrapper}>
          <Icon.Static
            name={iconName}
            size={24}
            color={
              disabled
                ? theme.palette['text-disabled']
                : theme.palette['text-primary']
            }
          />
        </View>
      )}

      <View style={[styles.content, contentStyle]}>
        <UIText
          typography={{ type: 'BODY', size: isRTL ? 'arBody1Semibold' : 'enBody1Semibold' }}
          color={
            disabled
              ? theme.palette['text-disabled']
              : theme.palette['text-primary']
          }
        >
          {title}
        </UIText>

        {subtitle && (
          <UIText
            typography={{ type: 'BODY', size: isRTL ? 'arBody2' : 'enBody2' }}
            color={
              disabled
                ? theme.palette['text-disabled']
                : theme.palette['text-secondary']
            }
          >
            {subtitle}
          </UIText>
        )}
      </View>
    </Container>
  );
};

// ============================================================
// STYLES
// ============================================================

const createStyles = (theme: FullTheme) =>
  StyleSheet.create({
    container: {
      flexDirection: 'row',
      alignItems: 'center',
      padding: theme.spaces[16],
      backgroundColor: theme.palette['surface-primary'],
      borderRadius: theme.border.radius[8],
      gap: theme.spaces[12],
    },
    containerRTL: {
      flexDirection: 'row-reverse',
    },
    containerDisabled: {
      opacity: 0.5,
    },
    iconWrapper: {
      width: wp(40),
      height: wp(40),
      borderRadius: theme.border.radius[100],
      backgroundColor: theme.palette['surface-secondary'],
      alignItems: 'center',
      justifyContent: 'center',
    },
    content: {
      flex: 1,
      gap: theme.spaces[4],
    },
  });
```

---

## Template 2: withTheme HOC Pattern

Use when the component is part of `dc-ui-toolkit` or needs `useInverseTheme` support.

```typescript
// [ComponentName].tsx
import React from 'react';
import { I18nManager, StyleSheet, View } from 'react-native';
import type { ViewStyle } from 'react-native';

import { Text as UIText } from '@dc-ui-toolkit';
import {
  CustomFunctionComponent,
  FullTheme,
  ThemeProps,
  commonProps,
  withTheme,
} from '@dc-ui-toolkit-theme';

// ============================================================
// TYPES
// ============================================================

export interface [ComponentName]Props extends commonProps {
  title: string;
  variant?: 'primary' | 'secondary';
  style?: ViewStyle;
}

// ============================================================
// COMPONENT (internal — not exported directly)
// ============================================================

const [ComponentName]Base: CustomFunctionComponent<[ComponentName]Props> = ({
  theme,
  isRtl,
  title,
  variant = 'primary',
  style,
  testID,
}) => {
  const isPrimary = variant === 'primary';
  const rtlLayout = I18nManager.isRTL;

  return (
    <View
      style={[
        createStyles(theme).container,
        isPrimary
          ? createStyles(theme).primary
          : createStyles(theme).secondary,
        rtlLayout && createStyles(theme).rtl,
        style,
      ]}
      testID={testID}
    >
      <UIText
        typography={{
          type: 'BODY',
          size: isRtl ? 'arBody1' : 'enBody1',
        }}
        color={
          isPrimary
            ? theme.palette['text-primary']
            : theme.palette['text-secondary']
        }
      >
        {title}
      </UIText>
    </View>
  );
};

// ============================================================
// STYLES
// ============================================================

const createStyles = (theme: FullTheme) =>
  StyleSheet.create({
    container: {
      padding: theme.spaces[16],
      borderRadius: theme.border.radius[8],
    },
    primary: {
      backgroundColor: theme.palette['surface-primary'],
    },
    secondary: {
      backgroundColor: theme.palette['surface-secondary'],
      borderWidth: theme.border.widths?.[1] ?? 1,
      borderColor: theme.palette['input-field-field-stroke-default'],
    },
    rtl: {
      alignItems: 'flex-end',
    },
  });

// ============================================================
// EXPORT — wrap with withTheme
// ============================================================

export const [ComponentName] = withTheme([ComponentName]Base, '[ComponentName]');
```

---

## Index File

Always create alongside the component:

```typescript
// [ComponentName]/index.ts
export { [ComponentName] } from './[ComponentName]';
export type { [ComponentName]Props } from './[ComponentName]';
```

---

## RTL Rules for Components

Every component that renders directional layout must handle RTL:

```typescript
// 1. Use I18nManager.isRTL for layout direction (after app restart)
const isRtlLayout = I18nManager.isRTL;

// 2. Use isRTL from useTranslation for typography variant
const { isRTL } = useTranslation(); // isRTL = language === 'ar'

// 3. Flip margin/padding directionally
marginLeft: isRtlLayout ? 0 : theme.spaces[8],
marginRight: isRtlLayout ? theme.spaces[8] : 0,

// 4. Flip icon position
flexDirection: isRtlLayout ? 'row-reverse' : 'row',

// 5. Switch typography variant
typography={{ type: 'BODY', size: isRTL ? 'arBody1' : 'enBody1' }}
```

---

## Props Conventions

| Pattern | Convention | Example |
|---------|-----------|---------|
| Boolean state | `is*` / `has*` prefix | `isDisabled`, `hasIcon`, `isLoading` |
| Events | `on*` prefix | `onPress`, `onChange`, `onDismiss` |
| Style overrides | `*Style` suffix | `containerStyle`, `contentStyle` |
| Content | Descriptive noun | `title`, `subtitle`, `description` |
| Testing | Always include | `testID?: string` |

### Callback Signatures

```typescript
onPress?: () => void;
onChange?: (value: string) => void;
onSelect?: (item: ItemType, index: number) => void;
onSubmit?: (data: FormData) => void;
```

---

## Composition Patterns

### With Children

```typescript
interface CardProps {
  children: React.ReactNode;
  style?: ViewStyle;
}

export const Card: React.FC<CardProps> = ({ children, style }) => {
  const styles = useThemedStyles(createStyles);
  return <View style={[styles.container, style]}>{children}</View>;
};
```

### Compound Component

```typescript
// SummaryCard.tsx
const SummaryCardContainer: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <View>{children}</View>
);

const SummaryCardTitle: React.FC<{ text: string }> = ({ text }) => {
  const { isRTL } = useTranslation();
  return (
    <UIText typography={{ type: 'HEADING', size: isRTL ? 'arH2' : 'enH2' }}>
      {text}
    </UIText>
  );
};

const SummaryCardBody: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <View>{children}</View>
);

export const SummaryCard = {
  Container: SummaryCardContainer,
  Title: SummaryCardTitle,
  Body: SummaryCardBody,
};

// Usage
<SummaryCard.Container>
  <SummaryCard.Title text={t('summary.title')} />
  <SummaryCard.Body>{content}</SummaryCard.Body>
</SummaryCard.Container>
```

---

## Performance

```typescript
// 1. Memoize the component if used in lists
export const [ComponentName] = React.memo<[ComponentName]Props>(({ title, onPress }) => {
  // ...
});

// 2. Memoize callbacks
const handlePress = useCallback(() => {
  onPress?.();
}, [onPress]);

// 3. Memoize expensive style calculations with RTL
const styles = useMemo(
  () => createStyles(theme, I18nManager.isRTL),
  [theme],
);

// 4. Never inline objects or functions in JSX
// BAD
<View style={{ padding: 16 }} onLayout={() => handleLayout()} />

// GOOD
<View style={styles.container} onLayout={handleLayout} />
```

---

## Component Quality Checklist

- [ ] No dc-ui-toolkit component is being duplicated
- [ ] Props interface exported alongside component
- [ ] `testID` prop included
- [ ] RTL handled — both typography variant AND layout direction
- [ ] `I18nManager.isRTL` used for layout, `isRTL` from `useTranslation` for typography
- [ ] `useThemedStyles` (or `withTheme`) for all styles
- [ ] No hardcoded colors, spacing, or dimensions
- [ ] `React.memo` applied if component is used in a list
- [ ] `useCallback` on all handler functions
- [ ] `useMemo` on computed/derived values
- [ ] `disabled` state handled (opacity or conditional styling)
- [ ] `isLoading` state handled if component has async actions
- [ ] `index.ts` barrel file created
- [ ] Component placed in correct location (screen / base / toolkit)
- [ ] Import aliases used (`@dc-ui-toolkit`, `@hooks`, etc.)

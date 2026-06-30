# /build-screen - Create New Screens

## Purpose
Create a complete, production-ready screen following the Digital Dirham Wallet architecture. Outputs: screen file, hook file, test file, navigation registration, and translation keys.

## Agent Handoff (Post-Build)

After the screen, hook, and scaffold are created, automatically trigger these agents in sequence:

1. **`rtl-reviewer` agent** — audits the new screen + hook for RTL compliance (typography variants, directional styles, Android LRM, translation keys)
2. **`test-writer` agent** — reads the implementation and writes comprehensive test coverage (render, interactions, loading, error states, hook unit tests) (write this once all our implementation is done with all the error resolved before starting this always ask if any issue or erorr still there then procced with test-writer)

Invoke them by saying:
> "Now run the rtl-reviewer on [ScreenName]Screen.tsx and use[ScreenName].ts"
> "Now run the test-writer on [ScreenName]Screen.tsx and use[ScreenName].ts"

Or if the user says "full build", run both agents automatically after generating the files.

---

## Pre-Build Checklist

Before writing a single line:
1. Confirm the screen name and its entry in `RootStackParamList`
2. Identify route params (if any)
3. Identify which toolkit components are needed — check `@dc-ui-toolkit` first
4. **Choose screen wrapper:** Does the design show a gradient? → `RadialGradientBackground`. Flat color? → `ScreenLayout`
5. **Choose amount input:** Simple display? → `Input.AmountInput`. User entry with dynamic font scaling? → `Input.CurrencyAmountInput`
6. **Choose card type:** Pressable card with icon/title/subtitle/arrow? → `Cards.ActionCard`. Transaction row? → `Cards.ActivityCard`. Wallet visual card? → `Cards.WalletCard`
7. Determine if any API calls are needed (which React Query hook?)
8. Determine if Redux state is read or written
9. Identify if RTL-specific layout changes are required

---

## File Structure

```
src/scenes/[FeatureName]/
├── [FeatureName]Screen.tsx          # Render only
├── hooks/
│   └── use[FeatureName].ts          # All logic
├── components/                       # Screen-specific UI (if needed)
│   └── [SubComponent].tsx
└── __tests__/
    └── [FeatureName]Screen.test.tsx
```

---

## Screen Template

```typescript
// [FeatureName]Screen.tsx
import React from 'react';
import { StyleSheet, View, ScrollView, FlatList } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { SystemBars } from 'react-native-edge-to-edge';
import { StackScreenProps } from '@react-navigation/stack';

import {
  Button,
  Header,
  RadialGradientBackground,
  SkeletonLoader,
  Text as UIText,
  // Add other toolkit components as needed
} from '@dc-ui-toolkit';
import { FullTheme } from '@dc-ui-toolkit-theme';
import { useThemedStyles } from '@hooks/useThemedStyles';
import { RootStackParamList } from '@navigation/types/navigation';
import { use[FeatureName] } from './hooks/use[FeatureName]';

type Props = StackScreenProps<RootStackParamList, '[FeatureName]'>;

const [FeatureName]Screen: React.FC<Props> = ({ route }) => {
  const styles = useThemedStyles(createStyles);
  const {
    theme,
    useDark,
    t,
    isRTL,
    isLoading,
    // ... all hook values
    handleSubmit,
    handleBack,
  } = use[FeatureName]({ route });

  return (
    <RadialGradientBackground
      colors={theme.gradients[useDark ? 'surface-bg-gradient-dark' : 'surface-bg-gradient-light'].colors}
      centerX="80%"
      centerY="21.68%"
      radiusX="91.11%"
      radiusY="82.48%"
      style={styles.gradient}
    >
      <SystemBars style={useDark ? 'light' : 'dark'} hidden={false} />
      <SafeAreaView style={styles.safeContainer}>
        <Header.Basic
          title={t('[featureName].title')}
          onBackPress={handleBack}
        />

        <SkeletonLoader isLoading={isLoading}>
          <ScrollView
            style={styles.scrollView}
            contentContainerStyle={styles.scrollContent}
            showsVerticalScrollIndicator={false}
            keyboardShouldPersistTaps="handled"
          >
            {/* Content — use dc-ui-toolkit components only */}
            <UIText
              typography={{ type: 'HEADING', size: isRTL ? 'arH1' : 'enH1' }}
            >
              {t('[featureName].heading')}
            </UIText>

            <UIText
              typography={{ type: 'BODY', size: isRTL ? 'arBody2' : 'enBody2' }}
              color={theme.palette['text-secondary']}
            >
              {t('[featureName].description')}
            </UIText>
          </ScrollView>
        </SkeletonLoader>

        <View style={styles.footer}>
          <Button.Button
            variant={{ type: 'PRIMARY' }}
            title={t('[featureName].cta')}
            onPress={handleSubmit}
            testID="submit-button"
          />
        </View>
      </SafeAreaView>
    </RadialGradientBackground>
  );
};

const createStyles = (theme: FullTheme) =>
  StyleSheet.create({
    gradient: { flex: 1 },
    safeContainer: { flex: 1 },
    scrollView: { flex: 1 },
    scrollContent: {
      flexGrow: 1,
      padding: theme.spaces[16],
      gap: theme.spaces[16],
    },
    footer: {
      padding: theme.spaces[16],
      gap: theme.spaces[12],
    },
  });

export default [FeatureName]Screen;
```

### Alternative Template — Flat Background Screen

For screens with a solid background color (no gradient), use `ScreenLayout`:

```typescript
// [FeatureName]Screen.tsx — flat background variant
import React from 'react';
import { StackScreenProps } from '@react-navigation/stack';

import {
  ScreenLayout,
  TabView,
  Section,
  // Add other toolkit components as needed
} from '@dc-ui-toolkit';
import { useThemedStyles } from '@hooks/useThemedStyles';
import { FullTheme } from '@dc-ui-toolkit-theme';
import { RootStackParamList } from '@navigation/types/navigation';
import { use[FeatureName] } from './hooks/use[FeatureName]';

type Props = StackScreenProps<RootStackParamList, '[FeatureName]'>;

const [FeatureName]Screen: React.FC<Props> = ({ route }) => {
  const styles = useThemedStyles(createStyles);
  const {
    theme,
    t,
    isRTL,
    tabs,
    activeTabIndex,
    handleTabChange,
    handleFilter,
  } = use[FeatureName]({ route });

  return (
    <ScreenLayout
      title={t('[featureName].title')}
      headerRight={
        <Icon.Interactive name="Funnel" type="TERTIARY" onPress={handleFilter} testID="filter-button" />
      }
      backgroundColor={theme.palette['surface-base']}
    >
      <TabView
        tabs={tabs}
        initialTabIndex={activeTabIndex}
        onTabChange={handleTabChange}
      >
        {[
          <FirstTabContent key="tab-0" />,
          <SecondTabContent key="tab-1" />,
        ]}
      </TabView>
    </ScreenLayout>
  );
};

const createStyles = (theme: FullTheme) =>
  StyleSheet.create({
    // styles for tab content
  });

export default [FeatureName]Screen;
```

---

## Hook Template

```typescript
// hooks/use[FeatureName].ts
import { useCallback, useEffect, useMemo, useState } from 'react';
import { I18nManager, Platform } from 'react-native';
import { RouteProp, useNavigation, useFocusEffect } from '@react-navigation/native';
import { StackNavigationProp } from '@react-navigation/stack';

import { useTheme } from '@dc-ui-toolkit-theme';
import { useTranslation } from '@hooks/useTranslation';
import { useToast } from '@hooks/useToast';
import { RootStackParamList } from '@navigation/types/navigation';
import { useAppDispatch, useAppSelector } from '@redux/hooks';
// import { useWallets } from '@api/hooks/useWallets'; // add API hooks as needed

type NavigationType = StackNavigationProp<RootStackParamList, '[FeatureName]'>;
type RouteProps = {
  route: RouteProp<RootStackParamList, '[FeatureName]'>;
};

export const use[FeatureName] = ({ route }: RouteProps) => {
  // ─── Framework hooks ─────────────────────────────────────────────
  const { t, isRTL } = useTranslation();
  const { useDark, theme } = useTheme();
  const navigation = useNavigation<NavigationType>();
  const dispatch = useAppDispatch();
  const { showToast } = useToast();

  // ─── Route params ─────────────────────────────────────────────────
  const { /* paramName */ } = route?.params ?? {};

  // ─── Redux state ──────────────────────────────────────────────────
  // const { selectedWallet } = useAppSelector(state => state.wallets);

  // ─── Server state (React Query) ───────────────────────────────────
  // const { data, isLoading, refetch } = useWallets();

  // ─── Local state ──────────────────────────────────────────────────
  const [isLoading, setIsLoading] = useState(false);
  const [formValue, setFormValue] = useState('');
  const [errors, setErrors] = useState<Record<string, string | undefined>>({});

  // ─── Focus effects ────────────────────────────────────────────────
  // useFocusEffect(useCallback(() => { refetch(); }, [refetch]));

  // ─── Derived state ────────────────────────────────────────────────
  const isSubmitDisabled = useMemo(() => {
    return !formValue.trim() || Object.values(errors).some(Boolean);
  }, [formValue, errors]);

  // ─── Handlers ─────────────────────────────────────────────────────
  const handleFormChange = useCallback((text: string) => {
    setFormValue(text);
    setErrors(prev => ({ ...prev, formValue: undefined }));
  }, []);

  const handleBack = useCallback(() => {
    navigation.goBack();
  }, [navigation]);

  const handleSubmit = useCallback(async () => {
    try {
      setIsLoading(true);
      // await apiCall(formValue);
      navigation.navigate('NextScreen' as any); // replace with correct route
    } catch {
      showToast(t('errors.generic'), 'error', 3000);
    } finally {
      setIsLoading(false);
    }
  }, [formValue, navigation, showToast, t]);

  // ─── Return ───────────────────────────────────────────────────────
  return {
    theme,
    useDark,
    t,
    isRTL,
    isLoading,
    formValue,
    errors,
    isSubmitDisabled,
    handleFormChange,
    handleBack,
    handleSubmit,
  } as const;
};
```

---

## Test Template

```typescript
// __tests__/[FeatureName]Screen.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';

// ─── Mocks ───────────────────────────────────────────────────────────
jest.mock('@dc-ui-toolkit', () => ({
  Button: { Button: 'Button' },
  Header: { Basic: 'Header.Basic' },
  Text: 'UIText',
  RadialGradientBackground: ({ children, style }: any) =>
    React.createElement('View', { style }, children),
  SkeletonLoader: ({ children }: any) => children,
  FullScreenLoader: 'FullScreenLoader',
}));

jest.mock('@dc-ui-toolkit-theme', () => ({
  useTheme: jest.fn(() => ({
    theme: {
      gradients: {
        'surface-bg-gradient-dark': { colors: ['#000', '#111'], angle: 45 },
        'surface-bg-gradient-light': { colors: ['#fff', '#f5f5f5'], angle: 45 },
      },
      palette: {
        'text-primary': '#000',
        'text-secondary': '#666',
        'surface-primary': '#fff',
        'surface-base': '#f5f5f5',
      },
      spaces: Array.from({ length: 50 }, (_, i) => i),
      border: { radius: Array.from({ length: 101 }, (_, i) => i) },
      globalStyles: {},
    },
    useDark: false,
  })),
}));

jest.mock('@hooks/useThemedStyles', () => ({
  useThemedStyles: jest.fn(creator =>
    creator({
      spaces: Array.from({ length: 50 }, (_, i) => i),
      border: { radius: Array.from({ length: 101 }, (_, i) => i) },
      palette: {},
    }),
  ),
}));

jest.mock('@hooks/useTranslation', () => ({
  useTranslation: jest.fn(() => ({
    t: jest.fn((key: string) => key),
    isRTL: false,
    language: 'en',
  })),
}));

jest.mock('@hooks/useToast', () => ({
  useToast: jest.fn(() => ({ showToast: jest.fn() })),
}));

jest.mock('react-native-edge-to-edge', () => ({ SystemBars: 'SystemBars' }));
jest.mock('react-native-safe-area-context', () => ({
  SafeAreaView: ({ children }: any) => children,
}));

jest.mock('@react-navigation/native', () => ({
  useNavigation: jest.fn(() => ({
    navigate: jest.fn(),
    goBack: jest.fn(),
  })),
  useFocusEffect: jest.fn(),
}));

jest.mock('@redux/hooks', () => ({
  useAppDispatch: jest.fn(() => jest.fn()),
  useAppSelector: jest.fn(() => ({})),
}));

// ─── Tests ────────────────────────────────────────────────────────────
const mockRoute = { params: {} };
const mockNavigation = { navigate: jest.fn(), goBack: jest.fn() };

describe('[FeatureName]Screen', () => {
  beforeEach(() => jest.clearAllMocks());

  it('renders without crashing', () => {
    const Screen = require('../[FeatureName]Screen').default;
    expect(() =>
      render(<Screen route={mockRoute} navigation={mockNavigation} />),
    ).not.toThrow();
  });

  it('renders page title', () => {
    const Screen = require('../[FeatureName]Screen').default;
    const { getByText } = render(<Screen route={mockRoute} navigation={mockNavigation} />);
    expect(getByText('[featureName].title')).toBeTruthy();
  });

  it('calls goBack when back is pressed', () => {
    const { useNavigation } = require('@react-navigation/native');
    const goBackMock = jest.fn();
    useNavigation.mockReturnValue({ navigate: jest.fn(), goBack: goBackMock });

    const Screen = require('../[FeatureName]Screen').default;
    // Trigger back press if applicable
  });

  // Add tests for:
  // - Loading state (SkeletonLoader shows when isLoading = true)
  // - Submit button disabled state
  // - Form interaction
  // - Error toast display
  // - Navigation on success
});
```

---

## Navigation Registration

### 1. Add to `RootStackParamList`

`src/navigation/types/navigation.ts`:

```typescript
export type RootStackParamList = {
  // ... existing
  [FeatureName]: {
    walletId?: string;
    amount?: number;
  } | undefined;
};
```

### 2. Register Screen

In the appropriate navigator (`AppNavigator.tsx` or equivalent):

```typescript
<Stack.Screen
  name="[FeatureName]"
  component={[FeatureName]Screen}
  options={{ headerShown: false }}
/>
```

---

## Translation Keys

`src/i18n/locales/en.json`:
```json
{
  "[featureName]": {
    "title": "Screen Title",
    "heading": "Screen Heading",
    "description": "Descriptive text for the screen.",
    "cta": "Confirm"
  }
}
```

`src/i18n/locales/ar.json`:
```json
{
  "[featureName]": {
    "title": "عنوان الشاشة",
    "heading": "عنوان رئيسي",
    "description": "نص وصفي للشاشة.",
    "cta": "تأكيد"
  }
}
```

---

## Quality Checklist

- [ ] `RadialGradientBackground` + `SystemBars` + `SafeAreaView` wrapper (gradient screens) OR `ScreenLayout` (flat-bg screens)
- [ ] All logic in `use[FeatureName].ts` — zero state/effects in screen
- [ ] Every handler wrapped in `useCallback`
- [ ] Every derived value wrapped in `useMemo`
- [ ] All text uses `<UIText>` with `typography` prop
- [ ] Typography switches between `en*` and `ar*` based on `isRTL`
- [ ] All spacing from `theme.spaces[n]`
- [ ] All colors from `theme.palette['...']`
- [ ] Loading state handled with `SkeletonLoader` or `FullScreenLoader`
- [ ] Error state handled with `Toast` via `useToast()`
- [ ] Hook returns `as const`
- [ ] `testID` props on all interactive elements
- [ ] Test file covers: render, interactions, loading, error
- [ ] Route added to `RootStackParamList`
- [ ] Screen registered in navigator with `headerShown: false`
- [ ] Translation keys added to both `en.json` and `ar.json`
- [ ] Import aliases used (no relative paths across features)
- [ ] No `console.log` statements
- [ ] No hardcoded values (colors, spacing, strings)

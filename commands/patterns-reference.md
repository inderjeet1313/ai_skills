# Wallet App Patterns Reference

Quick reference for all common patterns in the Digital Dirham Wallet application.

---

## Import Aliases

```typescript
@dc-ui-toolkit       → src/components/dc-ui-toolkit/components
@dc-ui-toolkit-theme → src/components/dc-ui-toolkit/theme
@dc-ui-toolkit-utils → src/components/dc-ui-toolkit/utils
@dc-ui-toolkit-hooks → src/components/dc-ui-toolkit/hooks
@hooks               → src/hooks
@scenes              → src/scenes
@navigation          → src/navigation
@redux               → src/redux
@utils               → src/utils
@constants           → src/constants
@assets              → src/assets
@api                 → src/api
```

---

## Screen Wrapper Pattern

### Pattern A — Gradient Background (standard screens)

Most screens use `RadialGradientBackground` from `@dc-ui-toolkit`:

```typescript
import React from 'react';
import { StyleSheet, View, ScrollView } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { SystemBars } from 'react-native-edge-to-edge';
import { RadialGradientBackground, Header } from '@dc-ui-toolkit';
import { FullTheme } from '@dc-ui-toolkit-theme';
import { useThemedStyles } from '@hooks/useThemedStyles';

const Screen = () => {
  const { theme, useDark } = useTheme();
  const styles = useThemedStyles(createStyles);

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
        <Header.Basic title={t('screen.title')} onBackPress={handleBack} />
        <ScrollView
          style={styles.scrollView}
          contentContainerStyle={styles.scrollContent}
          showsVerticalScrollIndicator={false}
          keyboardShouldPersistTaps="handled"
        >
          {/* Content — dc-ui-toolkit components only */}
        </ScrollView>
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
  });
```

### Pattern B — Flat Background (tab/activity screens)

For screens without a gradient (e.g., Activity, Dashboard tabs), use `ScreenLayout`:

```typescript
import { ScreenLayout, TabView, Section } from '@dc-ui-toolkit';

const Screen = () => {
  const { theme, t } = useScreen();
  return (
    <ScreenLayout
      title={t('screen.title')}
      headerRight={<Icon.Interactive name="Filter" type="TERTIARY" onPress={handleFilter} />}
      backgroundColor={theme.palette['surface-base']}
    >
      <TabView
        tabs={[t('tab.all'), t('tab.sent'), t('tab.received')]}
        initialTabIndex={0}
        onTabChange={handleTabChange}
      >
        {[<AllContent />, <SentContent />, <ReceivedContent />]}
      </TabView>
    </ScreenLayout>
  );
};
```

**Decision rule:** Use `RadialGradientBackground` when the design shows a radial gradient. Use `ScreenLayout` when the design shows a flat solid color background. Never use `LinearGradient` — use `RadialGradientBackground` instead.

---

## Custom Hook Pattern

Extract all screen logic to a custom hook:

```typescript
// hooks/useScreenName.ts
import { useCallback, useEffect, useMemo, useState } from 'react';
import { useNavigation } from '@react-navigation/native';
import { StackNavigationProp } from '@react-navigation/stack';

import { useTheme } from '@dc-ui-toolkit-theme';
import { useTranslation } from '@hooks/useTranslation';
import { RootStackParamList } from '@navigation/types/navigation';
import { useAppDispatch, useAppSelector } from '@redux/hooks';

type NavigationType = StackNavigationProp<RootStackParamList, 'ScreenName'>;

export const useScreenName = () => {
  // Hooks
  const { t } = useTranslation();
  const { useDark, theme } = useTheme();
  const navigation = useNavigation<NavigationType>();
  const dispatch = useAppDispatch();

  // State
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string>();

  // Redux
  const data = useAppSelector(state => state.slice.data);

  // Computed
  const computedValue = useMemo(() => {
    return data?.filter(item => item.active);
  }, [data]);

  // Effects
  useEffect(() => {
    // Initial load
  }, []);

  // Handlers
  const handleSubmit = useCallback(() => {
    // Logic
  }, []);

  const handleCancel = useCallback(() => {
    navigation.goBack();
  }, [navigation]);

  return {
    theme,
    useDark,
    t,
    loading,
    error,
    computedValue,
    handleSubmit,
    handleCancel,
  } as const;
};
```

---

## Themed Styles Pattern

```typescript
import { StyleSheet } from 'react-native';
import { FullTheme } from '@dc-ui-toolkit-theme';
import { useThemedStyles } from '@hooks/useThemedStyles';
import { wp, hp } from '@dc-ui-toolkit-utils';

// In component
const styles = useThemedStyles(createStyles);

// Style creator function
const createStyles = (theme: FullTheme) =>
  StyleSheet.create({
    container: {
      flex: 1,
      padding: theme.spaces[16],
      backgroundColor: theme.palette['surface-base'],
    },
    card: {
      padding: theme.spaces[16],
      borderRadius: theme.border.radius[8],
      backgroundColor: theme.palette['surface-primary'],
      gap: theme.spaces[12],
    },
    row: {
      flexDirection: 'row',
      alignItems: 'center',
      gap: theme.spaces[8],
    },
    icon: {
      width: wp(40),
      height: wp(40),
      borderRadius: theme.border.radius[100],
    },
  });
```

---

## Typography Usage

```typescript
import { Text as UIText } from '@dc-ui-toolkit';

// Always alias to avoid collision with RN Text
// Always switch en*/ar* based on isRTL
const { t, isRTL } = useTranslation();

// Display (64px, 48px, 40px)
<UIText typography={{ type: 'DISPLAY', size: isRTL ? 'arXL' : 'enXL' }}>XL Display</UIText>
<UIText typography={{ type: 'DISPLAY', size: isRTL ? 'arDisplay1' : 'enDisplay1' }}>Display 1</UIText>

// Heading (24px, 20px, 18px, 17px — Semibold)
<UIText typography={{ type: 'HEADING', size: isRTL ? 'arH1' : 'enH1' }}>Heading 1</UIText>
<UIText typography={{ type: 'HEADING', size: isRTL ? 'arH2' : 'enH2' }}>Heading 2</UIText>
<UIText typography={{ type: 'HEADING', size: isRTL ? 'arH3' : 'enH3' }}>Heading 3</UIText>
// enH4 (17px) — no Arabic equivalent, use only when design specifies
<UIText typography={{ type: 'HEADING', size: 'enH4' }}>Heading 4</UIText>

// Body (16px, 14px, 12px, 10px + Semibold/Medium variants)
<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody1' : 'enBody1' }}>Body 1</UIText>
<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody2' : 'enBody2' }}>Body 2</UIText>
<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody3' : 'enBody3' }}>Body 3</UIText>
<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody4' : 'enBody4' }}>Body 4</UIText>
<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody1Semibold' : 'enBody1Semibold' }}>Body 1 Bold</UIText>

// Label
<UIText typography={{ type: 'LABEL', size: isRTL ? 'arLabel1' : 'enLabel1' }}>Label 1</UIText>
<UIText typography={{ type: 'LABEL', size: isRTL ? 'arLabel2' : 'enLabel2' }}>Label 2</UIText>

// Link
<UIText typography={{ type: 'LINK', size: isRTL ? 'arLink1' : 'enLink1' }}>Link</UIText>

// DM Sans — for financial/numeric data ONLY, no RTL switching needed
<UIText typography={{ type: 'DISPLAY', size: 'dmSansBalance' }}>44px balance</UIText>
<UIText typography={{ type: 'LABEL', size: 'dmSansLabel' }}>DM Sans label</UIText>
<UIText typography={{ type: 'BODY', size: 'dmSansBody1Semibold' }}>DM Sans body</UIText>

// With custom color
<UIText
  typography={{ type: 'BODY', size: isRTL ? 'arBody2' : 'enBody2' }}
  color={theme.palette['text-secondary']}
>
  Secondary text
</UIText>
```

---

## Button Variants

```typescript
import { Button } from '@dc-ui-toolkit';

// Primary button (brown fill)
<Button.Button
  variant={{ type: 'PRIMARY' }}
  title="Primary Action"
  onPress={handlePress}
/>

// Secondary button (outlined)
<Button.Button
  variant={{ type: 'SECONDARY' }}
  title="Secondary Action"
  onPress={handlePress}
/>

// Tertiary button (text only)
<Button.Button
  variant={{ type: 'TERTIARY' }}
  title="Cancel"
  onPress={handlePress}
/>

// Tertiary with custom colors
<Button.Button
  variant={{
    type: 'TERTIARY',
    tertiaryColor: {
      pressedColor: theme.palette['text-secondary'],
      enabledColor: theme.palette['text-primary'],
      disabledColor: theme.palette['text-disabled'],
    },
  }}
  title="Custom Tertiary"
  onPress={handlePress}
/>

// With icon
<Button.Button
  variant={{ type: 'PRIMARY' }}
  title="With Icon"
  icon={{ name: 'ArrowRight', iconPosition: 'right' }}
  onPress={handlePress}
/>

// Loading state
<Button.Button
  variant={{ type: 'PRIMARY' }}
  title="Submit"
  isLoading={isSubmitting}
  onPress={handlePress}
/>

// Disabled state
<Button.Button
  variant={{ type: 'PRIMARY' }}
  title="Disabled"
  disabled={true}
  onPress={handlePress}
/>

// Back button
<Button.BackButtonPrimary onPress={() => navigation.goBack()} />

// Circular buttons
<Button.CircularPrimary icon="Plus" onPress={handlePress} />
<Button.CircularSecondary icon="Settings" onPress={handlePress} />
```

---

## Form Input Patterns

```typescript
import { Input } from '@dc-ui-toolkit';

// Basic input with label and validation
<Input.BasicInput
  label={t('form.field_label')}
  value={value}
  onChangeText={handleChange}
  onBlur={handleBlur}
  placeholder={t('form.placeholder')}
  isValid={!error}
  errorMessage={error}
/>

// With icon
<Input.BasicInput
  label={t('form.search')}
  value={searchQuery}
  onChangeText={setSearchQuery}
  icon={<Icon.Static name="MagnifyingGlass" size={20} />}
  iconPosition="left"
/>

// Amount input — dynamic font scaling (use for Transfer, Request, Gift, Refund, Amount screens)
// Supports ref for programmatic focus control
<Input.CurrencyAmountInput
  ref={inputRef}
  value={amount}
  onChangeText={handleAmountChange}
  onBlur={handleAmountFocusOut}
  error={amountError}
  editable={!isReadonly}
/>

// Simple amount display (use for simpler cases without dynamic font scaling)
<Input.AmountInput
  value={amount}
  onAmountChange={handleAmountChange}
  error={amountError}
/>

// Dropdown-style select input
<Input.SelectInput
  label={t('form.purpose_label')}
  placeholder={t('form.purpose_placeholder')}
  value={selectedValue?.description}
  onPress={handleSelectPress}
  isValid={!selectError}
  errorMessage={selectError}
/>

// Disabled input
<Input.BasicInput
  label={t('form.readonly')}
  value={readonlyValue}
  disabled={true}
/>
```

---

## ActionCard Pattern

```typescript
import { Cards } from '@dc-ui-toolkit';

// Standard pressable card (most common usage — settings, profile menu items)
<Cards.ActionCard
  iconName="ShieldCheck"
  title={t('profile.security')}
  subtitle={t('profile.security_subtitle')}
  rightIconName="CaretRight"
  onPress={handlePress}
  cardStyle={styles.actionCard}
/>

// Without right icon (non-navigational)
<Cards.ActionCard
  iconName="Bell"
  title={t('profile.notifications')}
  subtitle={t('profile.notifications_subtitle')}
  onPress={handlePress}
/>

// With custom icon JSX
<Cards.ActionCard
  icon={<View style={styles.customIcon}><UIImage image={someImage} /></View>}
  titleComponent={<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody1Semibold' : 'enBody1Semibold' }}>{title}</UIText>}
  subtitleComponent={<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody2' : 'enBody2' }}>{subtitle}</UIText>}
/>
```

---

## Section Pattern

```typescript
import { Section } from '@dc-ui-toolkit';

<Section
  title={t('dashboard.my_wallets')}
  count={wallets.length}
  countLabel={t('dashboard.wallets_count')}
>
  {wallets.map(wallet => (
    <Cards.WalletCard key={wallet.id} {...walletProps} />
  ))}
</Section>
```

---

## Loading State Patterns

```typescript
import { Skeleton, SkeletonLoader, FullScreenLoader } from '@dc-ui-toolkit';

// Full screen loader
{loading && <FullScreenLoader />}

// Skeleton for specific content
<SkeletonLoader>
  <Skeleton width="100%" height={hp(60)} borderRadius={8} />
  <Skeleton width="60%" height={hp(20)} borderRadius={4} />
</SkeletonLoader>

// Conditional rendering with skeleton
{loading ? (
  <ContentSkeleton />
) : (
  <ActualContent />
)}

// Pull-to-refresh
<ScrollView
  refreshControl={
    <RefreshControl
      refreshing={refreshing}
      onRefresh={handleRefresh}
    />
  }
>
  {content}
</ScrollView>
```

---

## Error State Patterns

```typescript
import { Error, Toast } from '@dc-ui-toolkit';

// Error component
<Error
  icon="AlertCircle"
  title={t('error.title')}
  description={t('error.description')}
  primaryButtonText={t('error.retry')}
  onPrimaryPress={handleRetry}
/>

// Inline error text
{error && (
  <Text
    typography={{ type: 'BODY', size: 'enBody2' }}
    color={theme.palette['error-error-text']}
  >
    {error}
  </Text>
)}

// Form validation errors
const [errors, setErrors] = useState<Record<string, string>>({});

<Input.BasicInput
  isValid={!errors.fieldName}
  errorMessage={errors.fieldName}
  // ...
/>
```

---

## Navigation Patterns

```typescript
import { useNavigation } from '@react-navigation/native';
import { StackNavigationProp } from '@react-navigation/stack';
import { RootStackParamList } from '@navigation/types/navigation';

type NavigationType = StackNavigationProp<RootStackParamList, 'CurrentScreen'>;

const navigation = useNavigation<NavigationType>();

// Simple navigation
navigation.navigate('TargetScreen');

// With params
navigation.navigate('TargetScreen', { paramName: value });

// Go back
navigation.goBack();

// Reset stack (common after auth)
navigation.reset({
  index: 0,
  routes: [{ name: 'Tabs' }],
});

// Replace current screen
navigation.replace('NewScreen', { params });

// Pop multiple screens
navigation.pop(2);
```

---

## Redux Patterns

```typescript
import { useAppDispatch, useAppSelector } from '@redux/hooks';

// Select state
const { data, loading, error } = useAppSelector(state => state.sliceName);

// Dispatch action
const dispatch = useAppDispatch();
dispatch(actionCreator(payload));

// In hook
useEffect(() => {
  dispatch(fetchData());
}, [dispatch]);

const handleSubmit = useCallback(() => {
  dispatch(submitData(formData));
}, [dispatch, formData]);
```

---

## Data Fetching Pattern

```typescript
import { useFetch } from '@api/useFetch';
import { API_PATHS } from '@constants/apiPaths';

// GET request
const { data, loading, error, fetch } = useFetch<ResponseType>({
  url: API_PATHS.get.endpoint,
  method: 'GET',
  lazy: true,  // Don't fetch on mount
  onSuccess: (data) => {
    // Handle success
  },
});

// Trigger fetch
useEffect(() => {
  fetch();
}, []);

// With path params
fetch({
  pathParams: { walletId: 'abc123' },
});

// With query params
fetch({
  params: { page: 1, limit: 10 },
});

// POST request
const { fetch: submitData, loading } = useFetch<ResponseType, RequestBody>({
  url: API_PATHS.post.endpoint,
  method: 'POST',
  onSuccess: handleSuccess,
});

submitData({ body: formData });
```

---

## Design Tokens Quick Reference

### Spacing
```typescript
theme.spaces[0]   // 0px
theme.spaces[2]   // 2px
theme.spaces[4]   // 4px
theme.spaces[8]   // 8px
theme.spaces[10]  // 10px
theme.spaces[12]  // 12px
theme.spaces[14]  // 14px
theme.spaces[16]  // 16px (standard)
theme.spaces[24]  // 24px
theme.spaces[36]  // 36px
theme.spaces[48]  // 48px
```

### Border Radius
```typescript
theme.border.radius[0]   // 0px (sharp)
theme.border.radius[4]   // 4px
theme.border.radius[8]   // 8px (cards)
theme.border.radius[12]  // 12px
theme.border.radius[16]  // 16px
theme.border.radius[20]  // 20px
theme.border.radius[24]  // 24px
theme.border.radius[100] // 100px (pill)
```

### Common Colors
```typescript
// Text
theme.palette['text-primary']
theme.palette['text-secondary']
theme.palette['text-disabled']

// Surfaces
theme.palette['surface-base']
theme.palette['surface-primary']
theme.palette['surface-secondary']

// Feedback
theme.palette['error-error-text']
theme.palette['success-success-text']
theme.palette['warning-warning-text']

// Inputs
theme.palette['input-field-field-fill-default']
theme.palette['input-field-field-stroke-default']
theme.palette['input-field-field-stroke-active']

// Buttons
theme.palette['buttons-and-selectors-brown-btn-default']
theme.palette['buttons-and-selectors-brown-btn-pressed']
```

---

## Fonts

| Language | Weights | Font Family | Usage |
|----------|---------|-------------|-------|
| English | Regular | Degular-Regular | Body text |
| English | Medium | Degular-Medium | Display, body medium |
| English | Semibold | Degular-Semibold | Headings, labels |
| Arabic | Regular | Noto Sans Arabic | Arabic body text |
| Arabic | Medium | Noto Sans Arabic Medium | Arabic display |
| Arabic | Semibold | Noto Sans Arabic SemiBd | Arabic headings |
| DM Sans | Medium | DMSans-Medium | Financial data, balances |
| DM Sans | Semibold | DMSans-SemiBold | Financial labels |
| DM Sans | Bold | DMSans-Bold | Financial headings |

---

## File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Screen | PascalCase + Screen | `DashboardScreen.tsx` |
| Hook | use + PascalCase | `useDashboardDetails.ts` |
| Component | PascalCase | `DashboardCard.tsx` |
| Test | Name + .test | `DashboardScreen.test.tsx` |
| Types | PascalCase | `DashboardTypes.ts` |
| Utils | camelCase | `formatCurrency.ts` |

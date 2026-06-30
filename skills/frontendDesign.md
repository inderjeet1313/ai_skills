# Frontend Design — Digital Dirham Wallet

Complete reference for building UI in the Digital Dirham Wallet React Native app. The design system is `dc-ui-toolkit` (`@dc-ui-toolkit`). Always check the toolkit first before writing any custom component.

---

## Component Selection Guide

### "What component do I use for...?"

| UI Need | dc-ui-toolkit Component |
|---------|-------------------------|
| Pressable card with icon + title + subtitle + chevron | `Cards.ActionCard` |
| Transaction / activity list row with initials + amount | `Cards.ActivityCard` |
| Visual wallet card with background image + balance | `Cards.WalletCard` |
| Generic padded container card | `Cards.Card` |
| Key-value detail table (review screens) | `Cards.TwoColumnContentList` |
| Transfer from/to display | `Cards.FromToTransferCard` |
| Alert / notice banner | `Cards.TransferReviewNotice` |
| Info display with label + value + action | `Cards.InfoCard` |
| Primary/secondary/tertiary action button | `Button.Button` |
| Round icon + label below (quick actions) | `Button.RoundIconLabel` |
| Circular icon button (solid) | `Button.CircularPrimary` |
| Circular icon button (outlined) | `Button.CircularSecondary` |
| Profile avatar / initials button | `Button.CircularProfile` |
| UAE Pass auth button | `Button.ThirdParty` |
| Radio selection button | `Button.RadioButton` |
| Back navigation button | `Button.BackButtonPrimary` |
| Text input with label + validation | `Input.BasicInput` |
| Amount entry with dynamic font scaling | `Input.CurrencyAmountInput` |
| Simple amount display input | `Input.AmountInput` |
| Dropdown / picker trigger | `Input.SelectInput` |
| Screen title + back button (gradient bg) | `Header.Basic` |
| Dashboard top header | `Header.DashboardHeader` |
| All text rendering | `Text as UIText` (never RN `Text`) |
| Formatted Dirham balance display | `CurrencyAmount` |
| Currency denomination card | `CurrencyCard` |
| Stacked currency cards | `CurrencyCardStack` |
| Animated tab underline selector | `Selectors.Tabs` |
| Horizontal pill group filter | `Selectors.PillGroup` |
| Show/hide balance toggle | `Selectors.BalanceSwitch` |
| Checkbox | `Selectors.Checkbox` |
| On/off animated switch | `Selectors.Toggle` |
| Card-based list item with icon + text | `Selectors.ListSelector` |
| Wallet chip selector | `Selectors.Wallets` |
| Transaction list row | `StackedLists.Transaction` + `mapTransactionToProps()` |
| Navigation list row | `StackedLists.NavigationList` |
| Balance row | `StackedLists.Balance` |
| Pending intent row | `StackedLists.Intent` |
| Radio option group | `StackedLists.StackedOptionGroup` |
| Bottom sheet | `BottomSheet` |
| Confirmation dialog | `ConfirmationDialog` |
| Blur overlay | `Overlay` |
| Horizontal section divider | `Dividers.Standard` |
| Full-screen spinner | `FullScreenLoader` |
| Skeleton wrapper for loading state | `SkeletonLoader` |
| Single skeleton block | `Skeleton` |
| Success/error/warning toast | `useToast()` → `showToast()` |
| Inline snackbar | `Snackbar.Standard` |
| Full-screen error page | `Error as UIError` |
| QR code display | `QRCodeCard` |
| Reference code with copy | `ReferenceCodeCard` |
| Wallet ID sharing | `WalletShare` |
| Card with copyable ID + icon | `DetailedContent` |
| Onboarding bottom sheet | `GetStartedSheet` |
| Multi-step tooltip | `CoachMark` |
| Instructions list | `InstructionsList` |
| Top-up / withdrawal method picker | `TopUpWithdrawMethods` |
| Purpose picker (two-level) | `PurposeBottomSheet` |
| Wallet info display | `WalletInfoDisplay` |
| Review/confirmation screen layout | `ReviewDetails` |
| Section header with count | `Section` |
| Flat-background screen wrapper | `ScreenLayout` |
| Tab container (content by tab) | `TabView` |
| Radial gradient background | `RadialGradientBackground` |

---

## Screen Wrapper Decision

```
Does the design show a radial gradient background?
├── YES → RadialGradientBackground + SystemBars + SafeAreaView + Header.Basic
└── NO  → ScreenLayout (flat solid color, built-in SafeAreaView + header)
```

### Gradient Screen (standard)
```tsx
<RadialGradientBackground
  colors={theme.gradients[useDark ? 'surface-bg-gradient-dark' : 'surface-bg-gradient-light'].colors}
  centerX="80%" centerY="21.68%" radiusX="91.11%" radiusY="82.48%"
  style={styles.gradient}
>
  <SystemBars style={useDark ? 'light' : 'dark'} hidden={false} />
  <SafeAreaView style={styles.safeContainer}>
    <Header.Basic title={t('screen.title')} onBackPress={handleBack} />
    <ScrollView style={styles.scrollView} contentContainerStyle={styles.scrollContent}
      showsVerticalScrollIndicator={false} keyboardShouldPersistTaps="handled">
      {/* content */}
    </ScrollView>
    <View style={styles.footer}>
      <Button.Button variant={{ type: 'PRIMARY' }} title={t('screen.cta')} onPress={handleSubmit} isLoading={isLoading} />
    </View>
  </SafeAreaView>
</RadialGradientBackground>
```

### Flat-Background Screen (Activity, tab screens)
```tsx
<ScreenLayout
  title={t('screen.title')}
  headerRight={<Icon.Interactive name="Funnel" type="TERTIARY" onPress={handleFilter} />}
  backgroundColor={theme.palette['surface-base']}
>
  <TabView tabs={[t('tab.all'), t('tab.sent')]} initialTabIndex={0} onTabChange={handleTabChange}>
    {[<AllContent />, <SentContent />]}
  </TabView>
</ScreenLayout>
```

**Never use `LinearGradient` from `react-native-linear-gradient` as a screen wrapper.**

---

## Typography System

### When to use which type

| Situation | Type | EN size | AR size |
|-----------|------|---------|---------|
| Page/modal title | HEADING | `enH1` | `arH1` |
| Section title | HEADING | `enH2` | `arH2` |
| Card title | HEADING | `enH3` | `arH3` |
| Sub-section title | HEADING | `enH4` | `enH4` (no ar*) |
| Primary body content | BODY | `enBody1` | `arBody1` |
| Secondary/description | BODY | `enBody2` | `arBody2` |
| Caption / metadata | BODY | `enBody3` | `arBody3` |
| Tiny annotations | BODY | `enBody4` | `arBody4` |
| Field labels | LABEL | `enLabel1` | `arLabel1` |
| Badge labels | LABEL | `enLabel2` | `arLabel2` |
| Inline links | LINK | `enLink1` | `arLink1` |
| Balance display (44px) | DISPLAY | `dmSansBalance` | `dmSansBalance` |
| Financial labels | LABEL | `dmSansLabel` | `dmSansLabel` |

### RTL switching rule
```tsx
// Standard EN/AR — always switch
<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody1' : 'enBody1' }} />

// DM Sans — NEVER switch (language-agnostic)
<UIText typography={{ type: 'DISPLAY', size: 'dmSansBalance' }}>{balance}</UIText>
```

### Color rule — always use palette
```tsx
<UIText
  typography={{ type: 'BODY', size: isRTL ? 'arBody2' : 'enBody2' }}
  color={theme.palette['text-secondary']}   // ✅ palette
/>
// NOT: color="#666"  ❌
```

---

## Theme Tokens

### Spacing
```
theme.spaces[0]   = 0px
theme.spaces[4]   = 4px
theme.spaces[8]   = 8px    (small gap)
theme.spaces[12]  = 12px   (button gap)
theme.spaces[16]  = 16px   (standard padding)
theme.spaces[24]  = 24px   (section gap)
theme.spaces[32]  = 32px
theme.spaces[48]  = 48px
```

### Border Radius
```
theme.border.radius[4]   = tight radius
theme.border.radius[8]   = cards, inputs
theme.border.radius[12]  = larger cards
theme.border.radius[16]  = sheets
theme.border.radius[20]  = rounded containers
theme.border.radius[24]  = heavily rounded
theme.border.radius[100] = pill / circle
```

### Semantic Palette Keys

| Key | Usage |
|-----|-------|
| `text-primary` | Main body text |
| `text-secondary` | Subdued / helper text |
| `text-subtle` | Placeholder / hint |
| `text-disabled` | Disabled state text |
| `text-inverse` | Text on dark surfaces |
| `surface-base` | App background |
| `surface-primary` | Cards, inputs |
| `surface-secondary` | Nested containers |
| `surface-tertiary` | Deeply nested |
| `error-error-text` | Error messages |
| `error-error-fill` | Error backgrounds |
| `success-success-text` | Success messages |
| `warning-warning-text` | Warning messages |
| `input-field-field-fill-default` | Input fill |
| `input-field-field-stroke-default` | Input border |
| `input-field-field-stroke-active` | Focused input border |
| `input-field-field-stroke-error` | Error input border |
| `buttons-and-selectors-brown-btn-default` | Primary button |
| `buttons-and-selectors-brown-btn-pressed` | Primary button pressed |

### Gradients
```tsx
theme.gradients['surface-bg-gradient-light']  // Light mode screen gradient
theme.gradients['surface-bg-gradient-dark']   // Dark mode screen gradient
// Access: .colors (array) — used in RadialGradientBackground
```

---

## Common Screen Compositions

### Dashboard Screen
```
ScreenLayout (or custom gradient header)
└── ScrollView
    ├── TotalBalanceCard (CurrencyAmount + BalanceSwitch)
    ├── Section (wallets)
    │   └── CurrencyCardStack or Cards.WalletCard[]
    ├── Section (quick actions)
    │   └── Button.RoundIconLabel[] (Send, Receive, Top Up, Withdraw)
    └── Section (recent activity)
        └── StackedLists.Transaction[]
```

### Activity / Tab Screen
```
ScreenLayout
└── TabView (tabs: Transactions, Analytics)
    ├── Tab 0: FlatList of Cards.ActivityCard
    └── Tab 1: Analytics charts
```

### Transfer / Amount Entry Screen
```
RadialGradientBackground
└── SafeAreaView
    ├── Header.Basic
    └── ScrollView
        ├── Input.CurrencyAmountInput (dynamic font scaling)
        ├── Input.SelectInput (purpose picker)
        └── Cards.TransferReviewNotice (if warning needed)
    └── Footer: Button.Button (PRIMARY)
```

### Review / Confirmation Screen
```
RadialGradientBackground
└── SafeAreaView
    ├── Header.Basic
    └── ReviewDetails (compound component)
        ├── Amount display
        ├── Cards.FromToTransferCard
        ├── Cards.TwoColumnContentList (fee, purpose, reference)
        ├── Cards.TransferReviewNotice (optional warning)
        └── Buttons (PRIMARY + TERTIARY)
```

### Settings / Profile Screen
```
RadialGradientBackground (or ScreenLayout)
└── SafeAreaView
    ├── Header.Basic
    └── ScrollView
        └── Cards.ActionCard[] (each setting row)
```

---

## Anti-Patterns — Never Do These

| Wrong | Right |
|-------|-------|
| `import { Text } from 'react-native'` | `import { Text as UIText } from '@dc-ui-toolkit'` |
| `import LinearGradient from 'react-native-linear-gradient'` | `import { RadialGradientBackground } from '@dc-ui-toolkit'` |
| Custom `<Pressable><Text>...</Text></Pressable>` card | `<Cards.ActionCard>` |
| Custom amount input with `Animated.Text` | `<Input.CurrencyAmountInput>` |
| Custom tab bar with manual state | `<TabView>` |
| Custom section header with count pill | `<Section>` |
| `style={{ color: '#333', fontSize: 16 }}` on any element | `theme.palette['text-primary']`, `UIText` with typography prop |
| Hardcoded spacing `padding: 16` | `padding: theme.spaces[16]` |
| `navigation.navigate(screen as never)` | Fix route in `RootStackParamList` |
| `useState` in a screen component | Move to `use[Feature].ts` hook |

---

## Design Quality Gates

Before submitting any UI work:

- [ ] Every string in `<UIText>` with `typography` prop (type + size)
- [ ] Typography size switches `en*`/`ar*` based on `isRTL` (except DM Sans variants)
- [ ] Zero hardcoded colors, spacing, border-radius values
- [ ] All icons from `Icon.Static` / `Icon.Interactive` / `Icon.StaticRound`
- [ ] Screen wrapper is `RadialGradientBackground` or `ScreenLayout` (not `LinearGradient`)
- [ ] Toolkit component used where one exists (check the Component Selection Guide above)
- [ ] `testID` on all interactive elements
- [ ] `useCallback` on all handlers, `useMemo` on all derived values
- [ ] RTL layout tested (margins/paddings flip, flex directions flip)
- [ ] Both `en.json` and `ar.json` updated with new translation keys

# /build-ui - UI Building Orchestrator

## Overview
Master skill for all UI work in the Digital Dirham Wallet React Native app. Analyzes the request and routes to the correct sub-skill while enforcing design system compliance at every step.

---

## Task Routing

When invoked, analyze the request and route:

| Request Type | Route To |
|---|---|
| Create a new screen | `/project:build-screen` |
| Create a screen-specific or shared component | `/project:build-component` |
| Add component to dc-ui-toolkit | `/project:add-to-toolkit` |
| Review existing UI/design compliance | `/project:design-review` |
| Quick pattern lookup | `/project:patterns-reference` |
| RTL compliance check | `rtl-reviewer` agent |
| Write/complete test files | `test-writer` agent |
| Code quality + architecture review | `code-reviewer` agent |
| Security audit | `security-analyzer` agent |

If unclear, ask: "Are you creating a screen, a component, reviewing existing code, or need a specialist review?"

## Agent Integration

These specialized agents are available and auto-trigger on relevant tasks:

| Agent | Triggers On |
|-------|------------|
| `rtl-reviewer` | "RTL", "Arabic", "typography variants", "ar.json", "I18nManager" |
| `test-writer` | "write tests", "test coverage", "test file", "add tests" |
| `code-reviewer` | "review code", "PR review", "code quality", "before merge" |
| `security-analyzer` | "security", "token", "auth flow", "payment security", "release audit" |

**Full build workflow (for a new screen):**
```
1. /project:build-screen     → screen + hook + scaffold
2. rtl-reviewer agent        → RTL compliance check
3. test-writer agent         → full test coverage
4. code-reviewer agent       → final quality gate
```

---

## DC-UI-TOOLKIT — ALWAYS CHECK FIRST

Before writing any UI code, verify no equivalent exists in dc-ui-toolkit. Import from `@dc-ui-toolkit`.

### Buttons

| Component | Props / Notes |
|---|---|
| `Button.Button` | `variant={{ type: 'PRIMARY' \| 'SECONDARY' \| 'TERTIARY' }}`, `title`, `onPress`, `isLoading`, `disabled`, `icon` |
| `Button.BackButtonPrimary` | Back navigation |
| `Button.CircularPrimary` | Round icon button — `variant: {type:'icon'}` or `{type:'logo'}`, `size: 'SMALL'|'MEDIUM'|'LARGE'|'XLARGE'` |
| `Button.CircularSecondary` | Secondary round icon button |
| `Button.CircularProfile` | Profile avatar button — shows image or initials fallback |
| `Button.ThirdParty` | UAE Pass / social auth |
| `Button.RadioButton` | Radio selection |
| `Button.RoundIconLabel` | Round icon with label below — `iconName`, `label`, `onPress`, `disabled?` (quick actions bar) |

### Text

```typescript
// Always alias to avoid collision with RN Text
import { Text as UIText } from '@dc-ui-toolkit';

// Required: typography prop with type + size
<UIText typography={{ type: 'HEADING', size: isRTL ? 'arH1' : 'enH1' }}>
  {t('key')}
</UIText>

// Optional: color, multiLine, numberOfLines, onPress, underline
<UIText
  typography={{ type: 'BODY', size: isRTL ? 'arBody2' : 'enBody2' }}
  color={theme.palette['text-secondary']}
  numberOfLines={2}
>
  {t('key')}
</UIText>
```

**Typography reference:**

| Type | EN sizes | AR sizes | Notes |
|------|----------|----------|-------|
| DISPLAY | enXL (64), enDisplay1 (48), enDisplay2 (40) | arXL, arDisplay1, arDisplay2 | |
| HEADING | enH1 (24), enH2 (20), enH3 (18), enH4 (17) | arH1, arH2, arH3 | enH4 has no ar* equivalent |
| BODY | enBody1 (16), enBody2 (14), enBody3 (12), enBody4 (10) + Semibold/Medium variants | arBody1-4 | |
| LABEL | enLabel1-3 + Semibold | arLabel1-3 | |
| LINK | enLink1-3 + Semibold | arLink1-3 | |
| DM_SANS | dmSansBalance (44), dmSansLabel, dmSansLabelSemibold, dmSansBody1Semibold, dmSansBody3Semibold, dmSansBody4Medium | — | Language-agnostic — **no RTL switching needed** |

> **DM Sans variants** are used for financial/numeric display (balances, amounts). They use DM Sans font. Do NOT apply `isRTL ? 'ar...' : 'en...'` switching to them.

### Inputs

```typescript
<Input.BasicInput
  label={t('field.label')}
  value={value}
  onChangeText={handleChange}
  isValid={!error}
  errorMessage={error}
  icon={<Icon.Static name="Scan" />}
  iconPosition="right"
  disabled={false}
/>

// Dynamic font-scaling amount input — use for Transfer, Request, Gift, Refund, Amount screens
// Supports ref for programmatic focus
<Input.CurrencyAmountInput
  ref={inputRef}
  value={amount}
  onChangeText={handleAmountChange}
  onBlur={handleAmountBlur}
  error={amountError}
  editable={!isReadonly}
/>

// Simple amount display (for cases not needing dynamic font scaling)
<Input.AmountInput
  value={amount}
  onAmountChange={handleAmountChange}
  error={amountError}
/>

// Dropdown-style select field
<Input.SelectInput
  label={t('purpose.label')}
  placeholder={t('purpose.placeholder')}
  value={selectedPurpose?.description}
  onPress={handlePurposePress}
  isValid={!purposeError}
  errorMessage={purposeError}
/>
```

### Cards

| Component | Usage |
|---|---|
| `Cards.Card` | Generic container card |
| `Cards.Account` | Wallet account — `heading`, `walletIcon`, `title`, `balance`, `errorText` |
| `Cards.ActionCard` | **Pressable card** — `iconName`, `title`, `subtitle`, `rightIconName`, `onPress`, `cardStyle` — use for settings rows, menu items, feature cards (6+ screens) |
| `Cards.ActivityCard` | **Transaction/activity row** — `title`, `subtitle`, `initials?`, `titleIcon`, `rightContent`, `onPress` |
| `Cards.InfoCard` | Info display — `label`, `value`, `actionIcon?`, `onAction?`, `footerText?` |
| `Cards.WalletCard` | **Visual wallet card** — `walletId`, `balance`, `isPrimary`, `backgroundImage`, `onPress`, `variant: 'default'\|'compact'` |
| `Cards.TransferReviewNotice` | Alert notice — `variant: 'info'\|'warning'\|'error'\|'success'`, `title`, `message` |
| `Cards.TwoColumnContentList` | Two-column details — `rows: {label, value, showCurrencyIcon?, onCopy?}[]`, `title?` |
| `Cards.FromToTransferCard` | Transfer from/to display |
| `CurrencyCard` | Single currency denomination card |
| `CurrencyCardStack` | Stacked currency cards (Dashboard) |

```typescript
// ActionCard — most common new card pattern (6+ screens)
<Cards.ActionCard
  iconName="ShieldCheck"
  title={t('profile.security')}
  subtitle={t('profile.security_subtitle')}
  rightIconName="CaretRight"
  onPress={handlePress}
  cardStyle={styles.card}
/>

// ActivityCard — transaction/activity rows
<Cards.ActivityCard
  title={counterparty}
  subtitle={subtitle}
  initials={initials}
  titleIcon={isCredit ? 'ArrowDownLeft' : 'ArrowUpRight'}
  rightContent={<UIText typography={{ type: 'BODY', size: 'enBody1Semibold' }}>{amount}</UIText>}
  onPress={handlePress}
/>

// WalletCard — visual wallet card with background
<Cards.WalletCard
  walletId={wallet.name}
  balance={formattedBalance}
  isPrimary={wallet.isPrimary}
  backgroundImage={wallet.backgroundImage}
  onPress={() => handleWalletPress(wallet.id)}
/>
```

### Headers

```typescript
<Header.Basic title={t('screen.title')} onBackPress={handleBack} />
<Header.DashboardHeader userName={name} />
```

### Icons

```typescript
// Static icon (display only)
<Icon.Static name="ArrowLineRight" size={24} color={theme.palette['text-primary']} />

// Interactive icon (tappable)
<Icon.Interactive name="CaretLeft" variant="PRIMARY" onPress={handleBack} />

// Icon with circular background
<Icon.StaticRound name="Plus" backgroundColor={theme.palette['surface-secondary']} />
```

**Common icon names:** ArrowLineRight, ArrowLineLeft, CaretLeft, CaretRight, CaretUp, CaretDown, Plus, Minus, Scan, Copy, Share, Gear, User, Bell, Lock, Eye, EyeSlash, Check, X, Info, Warning, Home, Wallet, History, Transfer, TopUp, Withdraw, QRCode, Search, Filter, Edit, Trash

**Bank wallet logos:** ADCBWalletLogo, ADIBWalletLogo, AjmanWalletLogo, AlansariWalletLogo, CBDWalletLogo, DIBWalletLogo, ENBDWalletLogo, FABWalletLogo, LuluExchangeWalletLogo, MashreqWalletLogo, RAKBankWalletLogo, SIBWalletLogo, UaePassLogo

### Lists & Stacked Components

```typescript
// Transaction item — ALWAYS use the helper
import { StackedLists, mapTransactionToProps } from '@dc-ui-toolkit';
<StackedLists.Transaction
  {...mapTransactionToProps(transaction, userWalletId, () => handleTxPress(transaction))}
/>

// Balance item
<StackedLists.Balance
  label={t('wallet.balance')}
  amount={balance}
  walletIcon={walletIcon}
  onPress={handlePress}
  variant="interactive"
/>

// Pending transaction row
<StackedLists.Intent transaction={intentTransaction} />

// Navigation menu item
<StackedLists.NavigationList
  label={t('profile.settings')}
  icon="Gear"
  onPress={handleSettingsPress}
  variant="interactive"
/>

// Radio group
<StackedLists.StackedOptionGroup
  title={t('options.title')}
  options={options}
  selected={selected}
  onSelect={handleSelect}
  showIcon
/>
```

### Selectors

```typescript
<Selectors.PillGroup
  options={[{ label: t('all'), value: 'all' }, { label: t('sent'), value: 'sent' }]}
  defaultValue="all"
  onChange={handleFilterChange}
/>
<Selectors.Tabs tabs={tabs} selectedIndex={activeIndex} onTabChange={setActiveIndex} />
<Selectors.BalanceSwitch
  showAmount={showBalance}
  onAmountToggle={toggleBalance}
  amount={balance}
  variant={cardVariant}
/>
<Selectors.Checkbox label={t('agree')} selected={checked} onPress={toggle} />
<Selectors.Toggle value={isEnabled} onValueChange={handleToggle} disabled={false} />
<Selectors.ListSelector text={t('topUp.visit_counter')} icon="Bank" />
<Selectors.Wallets wallets={wallets} selectedWalletId={selectedId} />
```

### Bottom Sheets & Overlays

```typescript
<BottomSheet isVisible={open} onDismiss={close} title={t('sheet.title')}>
  {/* content */}
</BottomSheet>

<ConfirmationDialog
  isVisible={dialogOpen}
  title={t('confirm.title')}
  message={t('confirm.message')}
  onConfirm={handleConfirm}
  onCancel={handleCancel}
/>

<Overlay isVisible={overlayOpen} onDismiss={closeOverlay} />
```

### Feedback

```typescript
// Toast — use useToast hook
const { showToast } = useToast();
showToast(t('success.saved'), 'success', 3000);
showToast(t('error.generic'), 'error', 3000);

// Snackbar
<Snackbar.Standard message={errorMessage} type="error" />

// Error state
<UIError message={t('error.load_failed')} onRetry={refetch} />
```

### Loaders & Skeletons

```typescript
// Full screen
<FullScreenLoader isVisible={isLoading} />

// Inline skeleton wrapper
<SkeletonLoader isLoading={loading}>
  <ActualContent />
</SkeletonLoader>

// Single skeleton block
<Skeleton width={wp(200)} height={hp(20)} borderRadius={theme.border.radius[8]} />
```

### Special Components

```typescript
<QRCodeCard value={walletId} size={200} />
<ReferenceCodeCard referenceCode={referenceCode} expiresIn={expiresIn} onCopy={handleCopy} />
<WalletShare walletId={walletId} icon={walletIcon} onClick={handleShare} />
<WalletInfoDisplay wallet={wallet} label={t('wallet.info')} />
<DetailedContent walletId={walletId} icon="IdentificationBadge" text={t('topUp.provide_proof')} />
<GetStartedSheet onClose={handleClose} />
<CoachMark steps={steps} visible={showCoach} onComplete={handleCoachComplete} />
<InstructionsList instructions={instructions} variant="default" />
<TopUpWithdrawMethods
  type="TOP-UP"
  validLFIs={validLFIs}
  selectedOption={selectedOption}
  onSelect={handleMethodSelect}
  onClose={closeSheet}
/>
<PurposeBottomSheet
  visible={purposeOpen}
  categories={categories}
  selectedPurpose={selectedPurpose}
  onSelect={handlePurposeSelect}
  onClose={closePurpose}
/>
```

### Layout Components

```typescript
import { ScreenLayout, TabView } from '@dc-ui-toolkit';

// ScreenLayout — flat-background screens (Activity, tab-level screens)
// ALTERNATIVE to RadialGradientBackground — use when design has no gradient
<ScreenLayout
  title={t('activity.title')}
  headerRight={
    <Icon.Interactive name="Funnel" type="TERTIARY" onPress={handleFilterPress} />
  }
  backgroundColor={theme.palette['surface-base']}
>
  <TabView
    tabs={[t('activity.transactions'), t('activity.analytics')]}
    initialTabIndex={0}
    onTabChange={handleTabChange}
  >
    {[<TransactionsTab />, <AnalyticsTab />]}
  </TabView>
</ScreenLayout>

// Quick actions row
<Button.RoundIconLabel
  iconName="ArrowUpRight"
  label={t('dashboard.send')}
  onPress={handleSendPress}
/>
```

### Section

```typescript
import { Section } from '@dc-ui-toolkit';

<Section
  title={t('dashboard.my_wallets')}
  count={wallets.length}
  countLabel={t('dashboard.wallets')}
>
  {wallets.map(wallet => (
    <Cards.WalletCard key={wallet.id} walletId={wallet.name} balance={balance} isPrimary={wallet.isPrimary} />
  ))}
</Section>
```

### Currency Display

```typescript
import { CurrencyAmount } from '@dc-ui-toolkit';

// Dashboard balance display
<CurrencyAmount
  amount={showBalance ? formattedBalance : '****'}
  variant="dashboard"
  color={theme.palette['text-primary']}
/>

// Wallet card balance
<CurrencyAmount amount={balance} variant="walletCard" />

// Compact wallet card
<CurrencyAmount amount={balance} variant="compactWalletCard" />
```

### Review Screen (Compound Component)

```typescript
import { ReviewDetails } from '@dc-ui-toolkit';

// Full review/confirmation screen layout
<ReviewDetails
  formattedAmount={formattedAmount}
  fromTo={{
    from: senderWalletId,
    to: recipientWalletId,
    fromIcon: senderIcon,
    toIcon: recipientIcon,
  }}
  detailRows={[
    { label: t('review.fee'), value: fee },
    { label: t('review.purpose'), value: purpose },
  ]}
  notice={hasWarning ? { variant: 'warning', title: t('review.warning'), message: warningMsg } : undefined}
  primaryAction={{ title: t('review.confirm'), onPress: handleConfirm, isLoading }}
  secondaryAction={{ title: t('review.cancel'), onPress: handleCancel }}
/>
```

---

## Required Imports Reference

```typescript
// Toolkit components — import only what you use
import {
  // Buttons
  Button,
  // Cards
  Cards,
  // Currency
  CurrencyCard, CurrencyCardStack, CurrencyAmount,
  // Dividers, Error
  Dividers, Error as UIError,
  // Loaders
  FullScreenLoader,
  // Headers
  Header,
  // Icons
  Icon,
  // Inputs
  Input,
  // Misc
  Jumbotron,
  // Gradient background
  RadialGradientBackground,
  // Selectors
  Selectors,
  // Skeleton
  Skeleton, SkeletonLoader,
  // Feedback
  Snackbar,
  // Lists
  StackedLists,
  // Text
  Text as UIText,
  // Toast
  Toast,
  // Overlays
  BottomSheet, ConfirmationDialog, Overlay,
  // Domain components
  WalletShare, CoachMark, DetailedContent, GetStartedSheet,
  InstructionsList, PurposeBottomSheet, QRCodeCard,
  ReferenceCodeCard, TopUpWithdrawMethods, TopUpWithdrawMethodsBottomSheet,
  WalletInfoDisplay,
  // New layout
  ScreenLayout, TabView, Section,
  // Review
  ReviewDetails,
  // Helpers
  mapTransactionToProps,
} from '@dc-ui-toolkit';

// Theme
import { useTheme, FullTheme } from '@dc-ui-toolkit-theme';
import { withTheme, CustomFunctionComponent, commonProps } from '@dc-ui-toolkit-theme';

// Utils
import { wp, hp, SCREEN_WIDTH, SCREEN_HEIGHT } from '@dc-ui-toolkit-utils';

// Hooks
import { useThemedStyles } from '@hooks/useThemedStyles';
import { useTranslation } from '@hooks/useTranslation';
import { useToast } from '@hooks/useToast';

// Navigation
import { NavigationProp, useNavigation, RouteProp, useFocusEffect } from '@react-navigation/native';
import { StackScreenProps, StackNavigationProp } from '@react-navigation/stack';
import { RootStackParamList } from '@navigation/types/navigation';

// Redux
import { useAppDispatch, useAppSelector } from '@redux/hooks';

// Screen wrapper
import { SafeAreaView } from 'react-native-safe-area-context';
import { SystemBars } from 'react-native-edge-to-edge';
```

---

## Screen Wrapper Pattern

### Primary — Gradient Background (most screens)

```typescript
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
    {/* Screen content */}
  </SafeAreaView>
</RadialGradientBackground>
```

### Alternative — Flat Background (Activity/tab screens)

```typescript
// Use when design shows a solid background color, not a radial gradient
<ScreenLayout
  title={t('screen.title')}
  headerRight={<Icon.Interactive name="Filter" type="TERTIARY" onPress={handleFilter} />}
  backgroundColor={theme.palette['surface-base']}
>
  {/* Screen content — often TabView */}
</ScreenLayout>
```

---

## RTL Checklist (Run on Every UI Task)

- [ ] `typography` size switches: `isRTL ? 'arBody1' : 'enBody1'`
- [ ] Directional margins/padding flip based on `I18nManager.isRTL`
- [ ] `flexDirection: 'row'` containers have RTL-aware layout
- [ ] Amount inputs handle `LRM` character on Android RTL
- [ ] Arabic translation keys exist for all strings

```typescript
// Get RTL context
const { isRTL } = useTranslation();         // for typography variants
const isRtlLayout = I18nManager.isRTL;      // for layout direction

// Typography
<UIText typography={{ type: 'BODY', size: isRTL ? 'arBody1' : 'enBody1' }} />

// Layout
flexDirection: isRtlLayout ? 'row-reverse' : 'row',
marginLeft: isRtlLayout ? 0 : theme.spaces[8],
marginRight: isRtlLayout ? theme.spaces[8] : 0,
```

---

## Coding Principles (Non-Negotiable)

### Screen = Render Only
```typescript
// WRONG
const Screen = () => {
  const [data, setData] = useState([]);
  return <View />;
};

// CORRECT
const Screen = () => {
  const { data, handlePress } = useScreen({ route });
  return <View />;
};
```

### No Hardcoded Values
```typescript
// WRONG
padding: 16, color: '#FFF', borderRadius: 8

// CORRECT
padding: theme.spaces[16],
color: theme.palette['surface-primary'],
borderRadius: theme.border.radius[8],
```

### Memoize Everything Passed as Prop
```typescript
const handlePress = useCallback(() => doThing(), [doThing]);
const filtered = useMemo(() => items.filter(isActive), [items]);
```

### Type-Safe Navigation
```typescript
// WRONG
navigation.navigate(screen as never);

// CORRECT — fix the type in RootStackParamList
navigation.navigate('Transfer', { walletId, amount });
```

---

## Toolkit Violation Protocol

Before creating any component:
1. Search `src/components/dc-ui-toolkit/components/index.ts`
2. If found → use it
3. If similar exists → ask: "Extend via `/add-to-toolkit`?"
4. If not found → ask: "Will this be used in 3+ places?"
   - Yes → "Add to toolkit via `/add-to-toolkit`?"
   - No → Create as screen/feature component

**Never silently create a duplicate of a toolkit component.**

---

## Global Quality Checklist

After any UI implementation:
- [ ] All toolkit components used where available
- [ ] `RadialGradientBackground` + `SystemBars` + `SafeAreaView` wrapper on gradient screens, OR `ScreenLayout` for flat-background screens
- [ ] All text via `<UIText>` with `typography` prop
- [ ] Typography switches `en*`/`ar*` based on `isRTL`
- [ ] Zero hardcoded colors, spacing, or border-radius values
- [ ] All icons from `Icon.Static` / `Icon.Interactive` / `Icon.StaticRound`
- [ ] Loading state: `SkeletonLoader` or `FullScreenLoader`
- [ ] Error state: `Toast` via `useToast()` or `UIError` component
- [ ] Screen logic in `use[FeatureName].ts` hook
- [ ] Hook returns `as const`
- [ ] Handlers in `useCallback`, derived values in `useMemo`
- [ ] `testID` on all interactive elements
- [ ] Import aliases used (no relative cross-feature paths)
- [ ] No `console.log` statements
- [ ] RTL layout and typography checked

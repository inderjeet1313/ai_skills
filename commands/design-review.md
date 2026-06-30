# /design-review - Review UI Code for Design System Compliance

## Purpose
Audit React Native code against the Digital Dirham Wallet engineering standards. Issues are categorized by severity. The review is **non-blocking** but Critical issues must be addressed before merging.

## Specialized Agent Delegation

`/design-review` handles UI/architecture compliance. For deeper specialist reviews, delegate to these agents:

| Need | Agent | How |
|------|-------|-----|
| Deep Arabic RTL audit | `rtl-reviewer` | "Use the rtl-reviewer on this file" |
| Full code quality review | `code-reviewer` | "Use the code-reviewer on this PR/feature" |
| Security audit | `security-analyzer` | "Use the security-analyzer on this flow" |
| Write missing tests | `test-writer` | "Use the test-writer on this screen" |

**Full pre-merge workflow:**
```
/design-review [files]          → UI/design compliance
code-reviewer agent [files]     → architecture + code quality
rtl-reviewer agent [files]      → Arabic RTL compliance
security-analyzer agent         → if touching auth, payments, or API
test-writer agent               → if test coverage is missing
```

---

## How to Run

Specify file(s) to review:
- A single screen: `/design-review TransferScreen`
- A hook: `/design-review useTransferScreen`
- A component: `/design-review WalletCard`
- A folder: `/design-review scenes/TopUpWithdraw`

The review reads the file(s), checks against all categories below, and produces a structured report.

---

## Review Categories

---

### Category 1: Toolkit-First Compliance

**Check for:**
- [ ] No `import { Text } from 'react-native'` — must use `Text as UIText` from `@dc-ui-toolkit`
- [ ] No custom button when `Button.Button` / `Button.CircularPrimary` / `Button.RoundIconLabel` exists
- [ ] No custom text input when `Input.BasicInput` exists
- [ ] No custom amount input when `Input.CurrencyAmountInput` (dynamic scaling) or `Input.AmountInput` (simple) exists
- [ ] No custom dropdown/select when `Input.SelectInput` exists
- [ ] No custom pressable card with icon+title+subtitle when `Cards.ActionCard` exists
- [ ] No custom transaction/activity row when `Cards.ActivityCard` exists
- [ ] No custom wallet visual card when `Cards.WalletCard` exists
- [ ] No custom section header with count when `Section` exists
- [ ] No custom tab container when `TabView` exists
- [ ] No custom flat-background screen wrapper when `ScreenLayout` exists
- [ ] No custom loading spinner — use `FullScreenLoader` or `SkeletonLoader`
- [ ] No duplicate implementation of any toolkit component
- [ ] Icons from `Icon.Static` / `Icon.Interactive` / `Icon.StaticRound` only
- [ ] `StackedLists.Transaction` + `mapTransactionToProps()` for transaction items
- [ ] Review/confirmation screen layout uses `ReviewDetails` where applicable

**Critical Violations:**
```typescript
// BAD — raw RN Text
import { Text } from 'react-native';
<Text style={{ fontSize: 24 }}>Send Money</Text>

// GOOD
import { Text as UIText } from '@dc-ui-toolkit';
<UIText typography={{ type: 'HEADING', size: 'enH1' }}>Send Money</UIText>
```

---

### Category 2: Screen Wrapper Structure

**Check for:**
- [ ] `RadialGradientBackground` (gradient screens) OR `ScreenLayout` (flat-bg screens) as outermost wrapper
- [ ] `SystemBars` with `style={useDark ? 'light' : 'dark'}` and `hidden={false}` (for RadialGradientBackground screens)
- [ ] `SafeAreaView` from `react-native-safe-area-context` (NOT from `react-native`) — used in RadialGradientBackground screens; ScreenLayout handles it internally
- [ ] `useDark` drives gradient key selection (RadialGradientBackground screens)
- [ ] `headerShown: false` on navigator

**Required Structure (gradient screens):**
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
    {/* content */}
  </SafeAreaView>
</RadialGradientBackground>
```

**Valid alternative (flat-background screens):**
```typescript
<ScreenLayout
  title={t('screen.title')}
  headerRight={/* optional */}
  backgroundColor={theme.palette['surface-base']}
>
  {/* content */}
</ScreenLayout>
```

**Critical:** `LinearGradient` (`react-native-linear-gradient`) is NOT a valid screen wrapper. Use `RadialGradientBackground` from `@dc-ui-toolkit` instead.

---

### Category 3: Typography

**Check for:**
- [ ] Every text node uses `<UIText>` with `typography` prop — no exceptions
- [ ] `typography` is `{ type: '...', size: '...' }` — not a string
- [ ] Typography variant switches between `en*` and `ar*` based on `isRTL`
- [ ] Correct type selected for context (HEADING for titles, BODY for content, LABEL for captions)
- [ ] `color` prop uses `theme.palette['...']` not hex

**Violations:**
```typescript
// CRITICAL — missing typography prop
<UIText>Hello</UIText>

// CRITICAL — wrong prop structure
<UIText typography="enH1">Hello</UIText>

// WARNING — not switching Arabic variant
<UIText typography={{ type: 'BODY', size: 'enBody1' }}>{t('key')}</UIText>
// Should be: size: isRTL ? 'arBody1' : 'enBody1'

// CRITICAL — hardcoded color
<UIText style={{ color: '#333' }}>Label</UIText>
```

**Typography Reference:**

| Type | EN Sizes | AR Sizes | Notes |
|------|----------|----------|-------|
| DISPLAY | enXL, enDisplay1, enDisplay2 | arXL, arDisplay1, arDisplay2 | |
| HEADING | enH1 (24px), enH2 (20px), enH3 (18px), enH4 (17px) | arH1, arH2, arH3 | enH4 has no ar* equivalent |
| BODY | enBody1-4 + Semibold + Medium variants | arBody1-4 | |
| LABEL | enLabel1-3 + Semibold | arLabel1-3 | |
| LINK | enLink1-3 + Semibold | arLink1-3 | |
| DM_SANS | dmSansBalance (44px), dmSansLabel, dmSansLabelSemibold, dmSansBody1Semibold, dmSansBody3Semibold, dmSansBody4Medium | — | Language-agnostic — no RTL switching |

> **Note:** DM Sans variants do NOT have Arabic equivalents. It is **correct** to use them without `isRTL` switching — they are for financial/numeric display only. Do NOT flag this as a violation.

---

### Category 4: Colors & Design Tokens

**Check for:**
- [ ] Zero hardcoded hex/rgb/rgba values
- [ ] All colors from `theme.palette['semantic-key']`
- [ ] All spacing from `theme.spaces[n]`
- [ ] All border radius from `theme.border.radius[n]`
- [ ] Uses `theme.globalStyles` shortcuts where available

**Violations:**
```typescript
// CRITICAL — hardcoded colors
backgroundColor: '#FFFFFF'
color: 'rgb(100, 100, 100)'
borderColor: 'rgba(0,0,0,0.1)'

// CRITICAL — hardcoded spacing
padding: 16
margin: 8
borderRadius: 8

// GOOD
backgroundColor: theme.palette['surface-primary']
padding: theme.spaces[16]
borderRadius: theme.border.radius[8]
```

**Semantic Color Guide:**
| Context | Palette Key |
|---------|-------------|
| Main text | `text-primary` |
| Subdued text | `text-secondary` |
| Placeholder/hint | `text-subtle` |
| Disabled | `text-disabled` |
| Error text | `error-error-text` |
| Error fill | `error-error-fill` |
| Success text | `success-success-text` |
| Warning text | `warning-warning-text` |
| Cards/inputs | `surface-primary` |
| App background | `surface-base` |
| Nested containers | `surface-secondary` |

---

### Category 5: Hook Pattern & Architecture

**Check for:**
- [ ] Screen file contains ONLY JSX return and `const styles = useThemedStyles(createStyles)`
- [ ] Zero `useState`, `useEffect`, `useCallback`, `useMemo` in screen files
- [ ] Hook file named `use[FeatureName].ts` (not `useLogic`, `useState`, `useScreen`)
- [ ] Hook returns `as const`
- [ ] All handlers wrapped in `useCallback` with correct dependency arrays
- [ ] All derived values wrapped in `useMemo`
- [ ] `useFocusEffect` used (not bare `useEffect`) for screen-focus refreshes
- [ ] Timer/interval cleanups returned from `useEffect`

**Violations:**
```typescript
// CRITICAL — logic in screen component
const TransferScreen = () => {
  const [amount, setAmount] = useState('');  // WRONG
  const wallets = useWallets();              // WRONG
  // ...
};

// CRITICAL — missing as const
return { theme, data, handlePress }; // WRONG
return { theme, data, handlePress } as const; // CORRECT

// WARNING — missing useCallback
const handlePress = () => navigation.goBack(); // WRONG
const handlePress = useCallback(() => navigation.goBack(), [navigation]); // CORRECT

// WARNING — missing cleanup
useEffect(() => {
  const timer = setTimeout(fn, 100);
  // MISSING: return () => clearTimeout(timer);
}, []);
```

---

### Category 6: RTL Implementation

**Check for:**
- [ ] `I18nManager.isRTL` used for layout direction (flex, margins, text alignment)
- [ ] `isRTL` from `useTranslation()` used for typography variant selection
- [ ] Directional styles flip correctly (marginLeft ↔ marginRight)
- [ ] `flexDirection: 'row'` components have RTL equivalent
- [ ] Amount inputs handle LRM character for Android RTL
- [ ] Arabic translation keys exist in `ar.json` for all `en.json` keys

**Violations:**
```typescript
// WARNING — typography doesn't switch variant
<UIText typography={{ type: 'BODY', size: 'enBody1' }}>{t('key')}</UIText>
// Should be: size: isRTL ? 'arBody1' : 'enBody1'

// WARNING — directional margin not flipped
marginLeft: theme.spaces[8],  // stays left in RTL
// Should be: marginLeft: isRTL ? 0 : theme.spaces[8], marginRight: isRTL ? theme.spaces[8] : 0

// WARNING — Android amount input missing LRM handling
// If this is an amount/numeric input used on Android, check for LRM (\u200E) pattern
```

---

### Category 7: State Management

**Check for:**
- [ ] Redux state not duplicating React Query server state (clear source of truth)
- [ ] No direct `AsyncStorage` calls outside of Redux persist config
- [ ] `useAppSelector` selector is targeted — not selecting entire slice
- [ ] Mutations via Redux dispatches or React Query `useMutation` (not mixed)
- [ ] No `as any` bypass on dispatch calls

**Violations:**
```typescript
// WARNING — selecting whole slice (over-subscription)
const state = useAppSelector(state => state.wallets); // BAD
const { selectedWallet } = useAppSelector(state => state.wallets); // GOOD

// WARNING — data duplication pattern
// Both Redux wallets AND React Query wallets read independently without sync
const reduxWallets = useAppSelector(s => s.wallets.wallets);
const { data: apiWallets } = useWallets();
// Either sync them or choose one source
```

---

### Category 8: API & React Query

**Check for:**
- [ ] All HTTP calls through `httpClient` from `@api/axiosInstance`
- [ ] Endpoint paths from `API_PATHS` — no inline URL strings
- [ ] Queries wrapped in custom hooks (`use[Resource].ts` in `src/api/hooks/`)
- [ ] `queryKey` includes all params that affect the query result
- [ ] `staleTime` set appropriately (not defaulting to 0 for frequently-hit endpoints)
- [ ] `useMutation` used for POST/PUT/DELETE, not `useQuery`
- [ ] Infinite scroll uses `useInfiniteQuery` with `getNextPageParam`

**Violations:**
```typescript
// CRITICAL — inline fetch
const response = await fetch('/api/wallets'); // WRONG

// CRITICAL — inline URL
await httpClient.get('/mobile/api/v1/wallets'); // WRONG
await httpClient.get(API_PATHS.get.getWallets); // CORRECT

// WARNING — queryKey missing params
queryKey: ['transactions'],      // WRONG if walletId or filters affect result
queryKey: ['transactions', walletId, filters], // CORRECT
```

---

### Category 9: Performance

**Check for:**
- [ ] Lists with 10+ items use `FlatList`, not `ScrollView`
- [ ] `FlatList.keyExtractor` returns stable unique ID (not array index)
- [ ] `renderItem` wrapped in `useCallback` or component is `React.memo`
- [ ] `removeClippedSubviews`, `maxToRenderPerBatch`, `windowSize` set on large lists
- [ ] No inline style objects in JSX — use `useThemedStyles`
- [ ] No inline function props — use `useCallback`
- [ ] `React.memo` on components rendered inside lists

**Violations:**
```typescript
// CRITICAL — FlatList with index key
keyExtractor={(_, index) => String(index)} // WRONG

// WARNING — inline style object (recreates every render)
<View style={{ padding: 16, backgroundColor: '#fff' }} />

// WARNING — inline function prop (re-renders children)
<Button onPress={() => handlePress(item.id)} />
// Should be: const handleItemPress = useCallback(() => handlePress(item.id), [item.id]);
```

---

### Category 10: Type Safety

**Check for:**
- [ ] No `as any` casts
- [ ] No `as never` navigation casts
- [ ] Route params typed in `RootStackParamList`
- [ ] API responses typed with interfaces
- [ ] Hook return type inferred via `as const`

**Violations:**
```typescript
// CRITICAL
navigation.navigate(screen as never);
dispatch(action as any);

// WARNING — untyped API response
const { data } = await httpClient.get('/wallets'); // should be httpClient.get<WalletResponse>(...)
```

---

### Category 11: Accessibility & Testing

**Check for:**
- [ ] `testID` on all interactive elements (buttons, inputs, list items)
- [ ] `testID` values are kebab-case and descriptive
- [ ] Touch targets are minimum 44×44 logical pixels
- [ ] Test file exists at `__tests__/[Name].test.tsx`
- [ ] Render test exists
- [ ] Key user interactions tested
- [ ] Loading and error states tested

---

### Category 12: Code Quality

**Check for:**
- [ ] No `console.log` / `console.error` outside `__DEV__` guards
- [ ] No commented-out code blocks
- [ ] No TODO comments older than current sprint
- [ ] Import aliases used (no `../../../hooks/...` style paths)
- [ ] No unused imports
- [ ] No magic string literals — use constants or translation keys

---

## Review Report Format

```markdown
## Design Review: [Screen/Component Name]
**File:** `src/scenes/[FeatureName]/[FeatureName]Screen.tsx`
**Reviewed:** [date]

### Summary
[2–3 sentences: overall quality, biggest concern, recommended priority.]

---

### Critical Issues (Must Fix Before Merge)

1. **[Issue Title]**
   - **File:** `path/to/file.tsx:42`
   - **Problem:** [What's wrong and why it matters]
   - **Fix:**
     ```typescript
     // Before
     <Text style={{ fontSize: 16 }}>Hello</Text>
     // After
     <UIText typography={{ type: 'BODY', size: isRTL ? 'arBody1' : 'enBody1' }}>Hello</UIText>
     ```

---

### Warnings (Fix in Follow-Up)

1. **[Issue Title]**
   - **File:** `path/to/file.tsx:87`
   - **Problem:** [What's wrong]
   - **Recommendation:** [How to improve]

---

### Suggestions (Optional Improvements)

1. [Suggestion] — `file.tsx:120`

---

### Compliance Scores

| Category | Score | Notes |
|----------|-------|-------|
| Toolkit-First Compliance | X/10 | |
| Screen Wrapper Structure | X/10 | |
| Typography | X/10 | |
| Colors & Design Tokens | X/10 | |
| Hook Pattern & Architecture | X/10 | |
| RTL Implementation | X/10 | |
| State Management | X/10 | |
| API & React Query | X/10 | |
| Performance | X/10 | |
| Type Safety | X/10 | |
| Accessibility & Testing | X/10 | |
| Code Quality | X/10 | |
| **Overall** | **X/10** | |
```

---

## Scoring Guide

| Score | Meaning |
|-------|---------|
| 10/10 | Fully compliant — no issues |
| 8–9/10 | Excellent — minor suggestions only |
| 6–7/10 | Good — warnings to address |
| 4–5/10 | Fair — multiple issues need fixing |
| 1–3/10 | Poor — critical issues present |
| 0/10 | Not implemented or completely wrong |

---

## Quick Flag Reference

### Red Flags (Critical)
- `import { Text } from 'react-native'`
- Hardcoded colors `'#FFF'`, `rgb(...)`, `rgba(...)`
- `as any` or `as never` type casts
- `useState` / `useEffect` directly inside a screen component
- Missing `RadialGradientBackground` + `SafeAreaView` OR `ScreenLayout` screen wrapper
- Using `LinearGradient` from `react-native-linear-gradient` as screen wrapper (use `RadialGradientBackground` instead)
- Duplicate implementation of a dc-ui-toolkit component (e.g., custom pressable card instead of `Cards.ActionCard`)
- Custom amount input with dynamic font instead of `Input.CurrencyAmountInput`
- `fetch()` or new `axios.create()` instead of `httpClient`
- Inline URL strings instead of `API_PATHS`

### Yellow Flags (Warning)
- Typography not switching `en*`/`ar*` based on `isRTL`
- Handlers missing `useCallback`
- Derived values missing `useMemo`
- `FlatList` missing stable `keyExtractor`
- `console.log` without `__DEV__` guard
- Missing timer cleanup in `useEffect`
- Import using relative path across feature boundaries
- Missing `testID` on interactive elements
- Hook not returning `as const`

### Blue Flags (Suggestion)
- Opportunity to extract repeated UI to a component
- Opportunity to extract shared logic to a hook
- Could use `theme.globalStyles` instead of re-declaring
- Component used in 3+ places — candidate for toolkit
- `React.memo` would help on this list item

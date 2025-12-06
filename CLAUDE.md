# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RetirementPlanner is a self-contained single-page web application for modeling retirement funds and determining tax-advantaged withdrawal strategies. The entire application is contained in `index.html` with no build process, dependencies, or external tooling required.

## Architecture

**Single-file application**: All HTML, CSS, and JavaScript exist in `index.html`. The application uses:
- Pure vanilla JavaScript (no frameworks)
- Chart.js (via CDN) for visualizations
- Google Fonts (Inter) for typography
- localStorage for persisting account data between sessions

**Dynamic account management**: The app uses a flexible account system:
- Users can add/remove any number of accounts for themselves and their spouse
- Each account has: type, current balance, and optional annual contribution
- Accounts are stored in the `accounts` array and persisted to localStorage
- Account types are defined in `accountTypes` object with metadata (category, RMD rules, etc.)

**Core calculation engine**: The app models:
1. Growth projections for multiple account types (401k, Roth IRA, HSA, taxable brokerage, SEP-IRA, rollover IRA, annuities)
2. Tax-optimized withdrawal strategies based on age, RMD requirements, and tax rates
3. Sequence of returns risk analysis over 5-year retirement windows
4. Dual-spouse retirement planning with separate retirement ages

**Key functions**:
- `renderAccounts()`: Dynamically generates UI for all user accounts
- `addAccount()` / `removeAccount()`: Manage the accounts array and sync to localStorage
- `getAccountDisplayName()`: Auto-labels accounts with #1, #2 suffixes when multiple of same type
- `calculateGrowth()`: Iterates through accounts array to project balances with contributions
- `calculateWithdrawalStrategy()`: Determines optimal withdrawal order to minimize taxes
- `calculateOptimalWithdrawals()`: Core logic for withdrawal sequencing based on tax efficiency
- `calculateDrawdown()`: Models portfolio sustainability under different market scenarios
- `getRetirementBalances()`: Groups accounts by owner and category, calculates future values

**Tax optimization logic**:
- Before age 73: Prioritizes taxable accounts (capital gains), then traditional accounts (to fill lower brackets), then HSA
- Age 73+: RMDs from traditional accounts are mandatory, supplemented by HSA, taxable, and finally Roth
- Account RMD calculations use IRS Uniform Lifetime Table divisors (stored in `rmdDivisors` object)
- Supports household planning with separate tracking for "your" accounts and spouse's accounts
- Tax categories: `traditional` (pre-tax with RMDs), `roth` (tax-free), `hsa` (triple advantage), `taxable` (capital gains)

## Development

**No build process**: Open `index.html` directly in a browser. All changes are immediately visible on refresh.

**Testing changes**: Since there's no test suite, manually verify calculations by:
1. Adjusting input values in the browser
2. Clicking "Calculate projection"
3. Reviewing outputs in stats cards, charts, and withdrawal timeline

**Browser compatibility**: Uses modern JavaScript (ES6+) and Chart.js v4. Test in Chrome, Firefox, Safari, or Edge.

## Code Modification Guidelines

**Adding new account types**:
1. Add the account type to the `accountTypes` object with appropriate metadata:
   - `category`: 'traditional', 'roth', 'hsa', or 'taxable'
   - `label`: Display name
   - `hasRMD`: Boolean indicating if RMDs apply
   - `rmdAge`: Age when RMDs begin (if applicable)
2. Add an option to the account type dropdown in the modal HTML
3. No other changes needed - the calculation functions automatically handle all account types by category

**Modifying account display/labeling**:
- Edit `getAccountDisplayName()` to change how accounts are labeled
- Currently uses "#1", "#2" suffixes for duplicate types per owner
- Accounts are grouped by owner ('self' vs 'spouse') in the UI

**Changing withdrawal logic**:
- The withdrawal order is defined in `calculateOptimalWithdrawals()` via the `phases` array
- Each phase has: age range, account type, priority, rationale, and tax rate
- Withdrawal strategy works with account categories, not specific types
- Modify the phases array to change strategy, or adjust the withdrawal logic in `createStrategyChart()`

**LocalStorage management**:
- `saveAccounts()` / `loadAccounts()` handle persistence
- Data is saved automatically on every account add/remove/update
- Clear localStorage to reset: `localStorage.removeItem('retirementAccounts')`

**Styling updates**:
- CSS custom properties define the color palette
- Uses CSS Grid for responsive layouts (`.main-grid`, `.stats-grid`, `.scenario-grid`)
- Account items styled with `.account-item` class
- Modal styled with `.modal` and `.modal-content` classes
- Follow existing Stripe-inspired design language (Inter font, subtle shadows, rounded corners)

## Important Data Structures

**Account object structure**:
```javascript
{
    id: 1,                          // Unique identifier
    owner: 'self' | 'spouse',       // Account owner
    type: '401k',                   // One of the accountTypes keys
    balance: 250000,                // Current balance
    annualContribution: 10000       // Annual contribution amount
}
```

**Account type definitions** (`accountTypes` object):
```javascript
'401k': {
    category: 'traditional',        // Tax treatment category
    label: '401k',                  // Display name
    hasRMD: true,                   // Subject to RMDs
    rmdAge: 73                      // Age when RMDs begin
}
```

**Market scenarios** (`marketScenarios` object):
- `below`: Bear market (-2.5% avg) based on 2000-2004, 2008-2012
- `average`: Normal market (10.2% avg) historical S&P 500
- `above`: Bull market (18.5% avg) based on 1995-1999, 2013-2017

**RMD divisors** (`rmdDivisors` object):
- IRS Uniform Lifetime Table for ages 73-87
- Used to calculate required minimum distributions
- Applied to traditional account categories (401k, Traditional IRA, SEP-IRA, Rollover IRA, Annuity)

**Account growth model**:
- Compounds annually: `(balance + contribution) * (1 + returnRate)`
- Each account is calculated independently
- Stops contributions at owner's retirement age (self vs spouse)
- Continues growth without contributions post-retirement
- Results are grouped by owner and tax category for withdrawal planning

## AI Integration

The "Get AI insights" feature (`getAIInsights()`) calls the Anthropic API directly from the browser. Note: This requires an API key to be embedded in the request, which is not production-ready for security reasons. The current implementation is incomplete (missing API key in headers).

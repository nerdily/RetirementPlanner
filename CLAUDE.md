# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RetirementPlanner is a self-contained single-page web application for modeling retirement funds and determining tax-advantaged withdrawal strategies. The entire application is contained in `index.html` with no build process, dependencies, or external tooling required.

## Architecture

**Single-file application**: All HTML, CSS, and JavaScript exist in `index.html`. The application uses:
- Pure vanilla JavaScript (no frameworks)
- Chart.js (via CDN) for visualizations
- Google Fonts (Inter) for typography

**Core calculation engine**: The app models:
1. Growth projections for multiple account types (401k, Roth IRA, HSA, taxable brokerage, SEP-IRA, rollover IRA, annuities)
2. Tax-optimized withdrawal strategies based on age, RMD requirements, and tax rates
3. Sequence of returns risk analysis over 5-year retirement windows
4. Dual-spouse retirement planning with separate retirement ages

**Key functions**:
- `calculateGrowth()`: Projects account balances from current age to retirement with contributions
- `calculateWithdrawalStrategy()`: Determines optimal withdrawal order to minimize taxes
- `calculateOptimalWithdrawals()`: Core logic for withdrawal sequencing based on tax efficiency
- `calculateDrawdown()`: Models portfolio sustainability under different market scenarios
- `getRetirementBalances()`: Calculates projected balances at each spouse's retirement age

**Tax optimization logic**:
- Before age 73: Prioritizes taxable accounts (capital gains), then traditional accounts (to fill lower brackets), then HSA
- Age 73+: RMDs from traditional accounts are mandatory, supplemented by HSA, taxable, and finally Roth
- Accounts RMD calculations use IRS Uniform Lifetime Table divisors (stored in `rmdDivisors` object)
- Supports household planning with separate tracking for "your" accounts and spouse's accounts

## Development

**No build process**: Open `index.html` directly in a browser. All changes are immediately visible on refresh.

**Testing changes**: Since there's no test suite, manually verify calculations by:
1. Adjusting input values in the browser
2. Clicking "Calculate projection"
3. Reviewing outputs in stats cards, charts, and withdrawal timeline

**Browser compatibility**: Uses modern JavaScript (ES6+) and Chart.js v4. Test in Chrome, Firefox, Safari, or Edge.

## Code Modification Guidelines

**Adding new account types**:
1. Add input fields in the HTML (following existing patterns in lines 595-659)
2. Update `getRetirementBalances()` to include the new account in growth calculations
3. Modify `calculateOptimalWithdrawals()` to incorporate the account into withdrawal strategy
4. Update chart datasets in `createStrategyChart()` if the account should be visualized

**Changing withdrawal logic**:
- The withdrawal order is defined in `calculateOptimalWithdrawals()` via the `phases` array
- Each phase has: age range, account type, priority, rationale, and tax rate
- Modify the phases array to change strategy, or adjust the withdrawal logic in `createStrategyChart()`

**Styling updates**:
- CSS custom properties (lines 16-34) define the color palette
- Uses CSS Grid for responsive layouts (`.main-grid`, `.stats-grid`, `.scenario-grid`)
- Follow existing Stripe-inspired design language (Inter font, subtle shadows, rounded corners)

## Important Data Structures

**Market scenarios** (`marketScenarios` object, lines 922-935):
- `below`: Bear market (-2.5% avg) based on 2000-2004, 2008-2012
- `average`: Normal market (10.2% avg) historical S&P 500
- `above`: Bull market (18.5% avg) based on 1995-1999, 2013-2017

**RMD divisors** (`rmdDivisors` object, lines 937-941):
- IRS Uniform Lifetime Table for ages 73-87
- Used to calculate required minimum distributions

**Account growth model**:
- Compounds annually: `(balance + contribution) * (1 + returnRate)`
- Stops contributions at retirement age for each spouse
- Continues growth without contributions post-retirement

## AI Integration

The "Get AI insights" feature (lines 1820-1868) calls the Anthropic API directly from the browser. Note: This requires an API key to be embedded in the request, which is not production-ready for security reasons. The current implementation is incomplete (missing API key in headers).

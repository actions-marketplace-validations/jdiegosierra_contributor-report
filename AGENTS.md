# AGENTS.md

TODO

## Project Overview

GitHub Action that evaluates PR contributor quality using objective GitHub metrics to combat AI-generated spam. Analyzes
user's contribution history and calculates a score (0-1000) based on PR merge rate, contributions to quality repos,
community engagement, and more.

## Common Commands

```bash
pnpm install          # Install dependencies
pnpm test             # Run tests
pnpm lint             # Run ESLint
pnpm bundle           # Format + package (run after changing src/)
pnpm run all          # Format, lint, test, coverage, and package
pnpm local-action     # Test action locally (requires .env file)
```

To run a single test file:

```bash
NODE_OPTIONS=--experimental-vm-modules NODE_NO_WARNINGS=1 pnpm exec jest __tests__/metrics/pr-history.test.ts
```

## Architecture

```text
src/
├── index.ts              # Entry point
├── main.ts               # Main action orchestration
├── types/                # TypeScript interfaces
│   ├── config.ts         # Configuration types
│   ├── metrics.ts        # Metric data structures
│   ├── scoring.ts        # Scoring result types
│   └── github.ts         # GitHub API types
├── config/               # Input parsing and defaults
│   ├── inputs.ts         # Parse action inputs
│   └── defaults.ts       # Default values and constants
├── api/                  # GitHub API client
│   ├── client.ts         # Octokit wrapper with rate limiting
│   ├── queries.ts        # GraphQL queries
│   └── rate-limit.ts     # Rate limit handling
├── metrics/              # Individual metric calculators
│   ├── pr-history.ts     # PR merge rate analysis
│   ├── repo-quality.ts   # Contributions to starred repos
│   ├── reactions.ts      # Comment reactions analysis
│   ├── account-age.ts    # Account age and consistency
│   ├── issue-engagement.ts # Issue engagement metrics
│   ├── code-review.ts    # Code review contributions
│   └── spam-detection.ts # Spam pattern detection
├── scoring/              # Score calculation
│   ├── engine.ts         # Main scoring aggregation
│   ├── decay.ts          # Time-based decay toward baseline
│   └── normalizer.ts     # Score normalization utilities
└── output/               # Output formatting
    ├── comment.ts        # PR comment generation
    └── formatter.ts      # Action output formatting
```

## Key Concepts

### Scoring System

- **Baseline**: 500/1000 (neutral)
- **Range**: 0-1000
- Each metric produces a normalized score (0-100) then applies weight
- Spam patterns apply additional penalties
- Scores decay toward baseline based on activity recency

### Testing Pattern (ESM Mocking)

```typescript
import { jest } from '@jest/globals'
import * as core from '../__fixtures__/core.js'

jest.unstable_mockModule('@actions/core', () => core)

const { myFunction } = await import('../src/module.js')
```

### Build Pipeline

- Rollup bundles `src/index.ts` → `dist/index.js`
- **Critical**: Always run `pnpm bundle` after modifying `src/` - the `dist/` directory must be committed

## Code Conventions

- Use `@actions/core` for logging (`core.debug()`, `core.info()`, etc.)
- Use `.js` extensions in imports (ESM requirement)
- Document functions with JSDoc comments
- Weights must sum to 1.0 for proper normalization

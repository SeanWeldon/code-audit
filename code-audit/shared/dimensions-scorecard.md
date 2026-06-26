# Code Review Dimensions

10-area assessment framework for code takeover evaluation. Dispatch an Explore subagent to review ALL source files across these dimensions.

## Dimensions

### 1. Build Health
- Can the app build? Check tsconfig, babel/metro config, dependency conflicts, missing types
- `@ts-nocheck` or `@ts-ignore` on critical files
- Dependency version conflicts (peer deps, React version mismatches)

### 2. Architecture Quality
- File organization, separation of concerns, component structure
- State management patterns (monolithic context vs split concerns)
- Singleton anti-patterns, circular dependencies
- Adapter/abstraction patterns (or lack thereof)

### 3. Backend/Database Integration
- Auth setup (real vs mock, fallback behavior)
- Database schema visibility (migrations in repo or black box?)
- Row-Level Security / access control policies
- Hardcoded URLs, connection strings, API endpoints

### 4. Navigation & Routing
- Route structure, deep linking, modal management
- Screen orientation handling
- Navigation state persistence

### 5. Feature Completeness
- What screens/features have real implementation vs scaffolding/placeholders?
- Measure: implemented, partially implemented, or missing
- Cross-reference against feature spec if provided

### 6. Code Quality
- Error handling patterns (silent catches, swallowed errors)
- Type safety (TypeScript strict mode bypasses)
- Memory leaks (uncleaned intervals, refs, listeners)
- Console logging in production code
- Hardcoded magic values

### 7. Security Concerns
- Exposed credentials (API keys, tokens in source)
- Encryption implementation quality (standard vs homemade)
- Input validation, PHI/PII handling
- Auth flow completeness (recovery, verification, session management)

### 8. Missing Infrastructure
- Test coverage (framework, test count, coverage %)
- CI/CD pipeline presence
- Error monitoring (Sentry, etc.)
- Build/deployment automation
- Documentation quality

### 9. Dependency Bloat
- Total dependency count vs actually imported
- Identify unused packages by grepping for imports
- Estimate bundle size impact of dead deps
- Native dependency overhead (build time, APK/IPA size)

### 10. Critical Bugs
- Latent crashes, data loss scenarios, race conditions
- Untested code paths in critical flows
- State corruption scenarios

## Output Format

For each dimension assign a letter grade (A through F) and risk level (Low, Moderate, High, Critical). Include specific file:line references where possible. Conclude with an overall grade and recommendation.

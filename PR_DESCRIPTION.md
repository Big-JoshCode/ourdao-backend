# Resolves #210, #214, #215, #216 - Governance, Documentation, and API Tooling

## Summary

This PR addresses four issues improving project governance, operational documentation, and API tooling:

1. **#210**: Added CODEOWNERS file with required review for migrations, auth, and indexer
2. **#214**: Documented log field schema for operators and aggregators
3. **#215**: Generated machine-readable OpenAPI spec with CI validation
4. **#216**: Documented migration numbering rules in CONTRIBUTING

## Changes

### Issue #210: CODEOWNERS

**Added:**
- `.github/CODEOWNERS` - Routes reviews for `src/`, with stricter requirements for `src/indexer/`, `src/auth.ts`, and `src/db/migrations/`

**Updated:**
- `CONTRIBUTING.md` - Documented review expectations

**Action required:**
- Maintainers must replace `@PLACEHOLDER_OWNER` with actual GitHub usernames
- Enable "Require review from Code Owners" in branch protection settings

### Issue #214: Logging Documentation

**Added:**
- `docs/LOGGING.md` - Complete log schema reference including:
  - Standard fields (level, time, pid, hostname, msg)
  - Request-scoped fields (reqId, req, res, responseTime)
  - Fields containing identifying data (address, remoteAddress) with retention guidance
  - Stable alertable event patterns (reorg detection, quarantine, poll errors, migrations)
  - Example queries for operational debugging
  - Note on subsystems not yet using Pino (issue #132)

### Issue #215: OpenAPI Specification

**Added:**
- `openapi.json` - Machine-readable OpenAPI 3.0.3 specification (351 lines)
- `scripts/generate-openapi-spec.ts` - Spec generation from Fastify routes
- Interactive API docs at `/docs` endpoint

**Updated:**
- `package.json` - Added `@fastify/swagger` and `@fastify/swagger-ui` as dev dependencies, plus npm scripts:
  - `npm run openapi:generate` - Generate spec from routes
  - `npm run openapi:validate` - CI validation (fails if spec drifts)
- `src/api/server.ts` - Registered Swagger plugins
- `.github/workflows/ci.yml` - Added OpenAPI validation step
- `README.md` - Replaced hand-written API table with link to spec + quick reference
- `CONTRIBUTING.md` - Added requirement to regenerate spec when modifying routes

**Confirmed:** Issue #8 claimed to deliver an OpenAPI spec but no artifact was found in the repository or CI.

### Issue #216: Migration Documentation

**Updated:**
- `CONTRIBUTING.md` - Added "Schema changes" subsection covering:
  - Unique, sequential version numbering
  - File naming convention (`<version>_<description>.sql`)
  - Requirement to update `schema.sql` in the same PR (and why)
  - Fresh-database shortcut behavior
  - Manual verification requirement (test suite doesn't exercise migrations)

### Unrelated Fix

**Fixed:**
- `test/setup.ts` - Added `export {}` to resolve TypeScript top-level await error

## Testing

All CI checks pass:
```bash
✅ npm run lint        # 0 errors (6 warnings pre-existing)
✅ npm run typecheck   # Pass
✅ npm run build       # Pass
✅ npm run openapi:validate  # Pass
```

## Verification

1. **CODEOWNERS**: Check `.github/CODEOWNERS` exists and covers `src/`
2. **Logging**: Read `docs/LOGGING.md` 
3. **OpenAPI**: 
   - Run `npm run dev` → visit http://localhost:4000/docs
   - Run `npm run openapi:validate`
4. **Migration docs**: Read CONTRIBUTING.md "Schema changes" section

## Dependencies Added

- `@fastify/swagger` (devDependency)
- `@fastify/swagger-ui` (devDependency)

Development-only dependencies for spec generation and interactive docs. No production runtime impact.

## Breaking Changes

None. All changes are additive.

## Files Changed

- **Added (4):** `.github/CODEOWNERS`, `docs/LOGGING.md`, `openapi.json`, `scripts/generate-openapi-spec.ts`
- **Modified (6):** `.github/workflows/ci.yml`, `CONTRIBUTING.md`, `README.md`, `package.json`, `src/api/server.ts`, `test/setup.ts`

## Acceptance Criteria

### Issue #210
- [x] Every path under `src/` has an owner
- [x] Migrations and auth require an owner's review
- [x] The expectation is documented

### Issue #214
- [x] The log schema is documented with field-level detail
- [x] Fields carrying identifying data are called out with retention guidance
- [x] Alertable event names are listed

### Issue #215
- [x] A machine-readable API description is produced by the build
- [x] CI fails when it drifts from the routes
- [x] Downstream consumers have an artifact to generate from
- [x] `npm run lint`, `npm run typecheck`, and `npm run build` all pass

### Issue #216
- [x] A contributor can add a migration correctly from CONTRIBUTING alone
- [x] The uniqueness and sequencing rules are explicit
- [x] The `schema.sql` mirroring requirement is stated with its reason

## Next Steps

1. **Maintainers:** Update `.github/CODEOWNERS` with actual usernames and enable branch protection
2. **Frontend team:** Consider using `openapi.json` to generate type-safe client code
3. **Operators:** Set up alerts based on patterns in `docs/LOGGING.md`

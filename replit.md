# Denge — Beslenme Takibi

Türkiye'deki yetişkin danışanlar ve diyetisyenler için beslenme takibi, izinli veri paylaşımı ve plan yönetimi sunan iOS/Android MVP prototipi.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Authentication and database settings are managed through Replit Secrets; never copy their values into code or logs.

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5, Clerk JWT authentication
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Mobile: Expo with Clerk email/password sign-in and secure token storage
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/beslenme-takip/` — Expo uygulaması.
- `artifacts/beslenme-takip/state/RemoteAppState.tsx` — authenticated API-backed mobile state.
- `artifacts/beslenme-takip/components/PilotScreens.tsx` — pilot client and dietitian screens.
- `docs/SRS-Turkiye.md` — ürün gereksinimleri ve Türkiye pazarı öncelikleri.
- `docs/pilot-operations.md` — uzmanların manuel doğrulanıp listeye alınması ve pilot operasyon sınırları.
- `lib/api-spec/openapi.yaml` — API contract and source for generated clients/validators.
- `lib/db/src/schema/` — profiles, nutrition logs, water totals, connections, and plans.

## Architecture decisions

- Clerk authenticates users; the API derives the acting user from the verified Clerk session and does not accept a client-selected role.
- New profiles default to `client`. A dietitian account must be manually reviewed, assigned the `dietitian` role, and listed as verified before it appears in discovery or receives client data. No self-service verification/admin workflow exists yet.
- The pilot currently supports email/password authentication; social sign-in is not configured.
- Dietitian connections record consent version, timestamp, and explicit scopes. Food logs, water data, and dietitian plans are gated by an active connection and the corresponding scope; clients can revoke access.
- The profile action deletes nutrition records and revokes connections but does not delete the Clerk account.
- Existing AsyncStorage data under `denge.mobile-demo.v1` is not uploaded or deleted automatically. The current pilot UI uses the server; a separate, explicit import flow is not implemented.
- Logs are manually entered. The app is not a medical device or a source of clinical advice. Final KVKK/privacy wording, retention policy, and professional verification procedure need legal and operational review before a public launch.

## Product

- Client: account-based daily food/macro records, water tracking, verified dietitian discovery, scope-based sharing consent, and plan viewing.
- Dietitian: view only consenting clients' allowed records and send plans when the client granted the plan scope.
- No sample dietitians or role-switch control are available in the pilot.
- iOS/Android için Expo; Türkiye odaklı Türkçe arayüz.

## User preferences

- Yanıtları Türkçe ver.

## Gotchas

- The server blocks future reads after a connection is revoked, but data already fetched can remain in an active screen's in-memory cache until that view refreshes or closes.
- The old local demo storage remains on the device and is not visible in the new server-backed UI.
- The directory intentionally stays empty until an operator manually verifies and lists a real dietitian.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details

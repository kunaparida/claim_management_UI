# Claim Management UI

Angular 22 front end for the claims backend in `chubb-claims-backendService`. **One app, three roles:**

| Role | Can do |
|---|---|
| **Claimant** | Register, report an incident, track their claims, answer an officer's request for information, read the decision |
| **Claims officer** | Work queue (oldest first), pick up claims, start review, request information, approve / reject, mark settled |
| **Manager** | Dashboard (open claims, liability per market, officer performance), see all claims, assign claims to officers |

## Run it locally

You need **Node 20+** and the backend running on **http://localhost:8080**.

1. Start the backend (JDK 21, Maven) from `chubb-claims-backendService`:
   ```bash
   mvn spring-boot:run
   ```
2. Start this app:
   ```bash
   npm install
   npm start
   ```
3. Open **http://localhost:4200**.

`npm start` proxies `/api` to the backend (`proxy.conf.json`). That is required, not optional: the backend has no CORS configuration, so
a browser cannot call it directly from another port.

**Accounts.** The backend seeds `officer@claims.local` and `manager@claims.local` (password `ChangeMe123!`) — the login page has one-click
buttons for them. Claimants register themselves (**Create an account**).

## Scripts

```bash
npm run build                  # production build into dist/
npm test -- --no-watch         # Vitest unit tests (216)
```

## How it is organised

See [`CLAUDE.md`](CLAUDE.md) for the backend contract, the architecture rules and the file layout, and [`AI_JOURNAL.md`](AI_JOURNAL.md) for
how it was built with AI — what was asked, what was accepted, what was corrected, and what was and was not verified.

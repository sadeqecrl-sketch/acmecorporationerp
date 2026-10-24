# company app Netlify + Supabase starter

This is a hosted web foundation for sign-in, company workspaces, scoped roles, request tracking, and audit entries. It is **not a complete production ERP**. Accounting ledgers and IFRS reports, Bangladesh tax/payroll configuration, POS connections, and migration of the local app's data still require separate implementation and review.

## 1. Create the Supabase project
1. Create a project and keep its database password private.
2. In **SQL Editor**, run the files in order:
   - supabase/migrations/0001_northstar_workspace.sql
   - supabase/migrations/0002_auth_hardening.sql
   - supabase/migrations/0003_company_profile.sql
3. In **Project Settings → API**, copy the Project URL and the publishable key (or legacy anon key). Never use the secret/service-role key in browser variables.

If you already ran 0001 and 0002, run only migration 0003. Do not rerun an already-applied migration.

## 2. Configure secure sign-in
1. In **Authentication → URL Configuration**, set the Site URL to your future Netlify address. Add that exact address and its wildcard path to allowed redirect URLs.
2. Keep email confirmation enabled. Configure a custom SMTP sender before relying on sign-up verification or password reset in production; Supabase's default email service is for limited testing.
3. In **Authentication → Providers → Email**, set a minimum password length of 12 or more and enable leaked-password protection where your Supabase plan supports it. Keep Auth rate limits enabled.
4. TOTP authenticator MFA must be enabled in Supabase Auth. The app requires it for owner, administrator, and auditor roles; the database policies also enforce the elevated-role MFA requirement.
5. Set VITE_ALLOW_INITIAL_SIGNUP to true for the first owner setup only. After the owner has verified their email, created the workspace, and enrolled an authenticator, change this value to false and redeploy. This prevents public self-registration. Staff accounts should be invited through Supabase Auth and assigned a least-privilege role in company app.

## 3. Deploy to Netlify
1. Upload this folder to a private Git repository and choose **Add new site → Import an existing project** in Netlify. Or build locally with Node.js using npm install and npm run build, then deploy the generated dist folder.
2. In Netlify **Site configuration → Environment variables**, add:
   - VITE_SUPABASE_URL = your Supabase Project URL
   - VITE_SUPABASE_ANON_KEY = your Supabase publishable/anon key
   - VITE_ALLOW_INITIAL_SIGNUP = true only during first-owner setup; false afterward
   - VITE_COMPANY_NAME = your company name shown on the sign-in screen before sign-in
3. Deploy. Environment variables are embedded at build time, so run a new deploy after changing them.
4. Sign up as the initial owner, verify the email, create the workspace, and follow the authenticator setup prompt. Then disable initial sign-up and deploy again.

## 4. Company information, staff, and account recovery
Owners and administrators can open Company profile in the app to edit the company name, street address, city, country, business email, phone, and website. This updates the workspace branding after sign-in. Set VITE_COMPANY_NAME in Netlify for the company name shown before sign-in on new devices.
Invite staff from **Supabase Authentication → Users**. After the account is confirmed, the owner/admin assigns the user's role from company app. Do not create shared accounts. Keep access limited to each person's job.

For a forgotten password, the sign-in page sends a reset link without revealing whether an email is registered. Configure the Netlify URL in Supabase allowed redirects and configure SMTP. Password reset links only work while valid and after the user opens the email link. Never ask staff to share their reset link or authenticator code.

If an owner loses their authenticator, use the organization's documented recovery process and verify the person through a separate trusted channel before an authorized Supabase project administrator resets MFA. Keep a second trusted authenticator available for privileged accounts. Do not disable MFA or RLS as a shortcut.

## Security and deployment notes
- The publishable/anon key is meant to be visible in the browser; Row Level Security is the database security boundary. The service-role key must never go into Netlify's VITE variables or client code.
- Netlify headers add HTTPS transport, anti-framing, MIME, referrer, permissions, and content-security protections. Review the Content Security Policy if adding another trusted integration.
- This browser-only starter uses the Supabase JavaScript client's persisted session. Protect signed-in devices, keep the browser current, and deploy only trusted code. A higher-assurance enterprise deployment can move sessions behind a server-side, HttpOnly-cookie application layer.
- Use MFA for project administrators, restricted invitations, HTTPS, backups, and a tested restore process.
- Request descriptions are not for passwords, access tokens, bank details, health information, or highly sensitive HR data.
- Bangladesh is used for generated request-ID calendar years only. It does not configure local VAT, tax, statutory payroll, or IFRS accounting. Have a qualified local accountant review those requirements.
- This starter does not connect ERPNext, Square, Shopify, Lightspeed, Dropbox, or OneDrive. Existing local data is not imported. Plan a reviewed migration and authenticated server-side provider connectors before using this as the live system of record.

## Contents
- src: web sign-in and workspace interface
- supabase/migrations: workspace schema, role policies, request IDs, authentication hardening, and company profile
- netlify.toml: build, routing, and web security headers
- .env.example: variable names only; no credentials




# VarsityStay — applications, photos and viewings

## Replace frontend files
Replace `src/main.js` and `src/style.css` with the files from this ZIP. Back up the originals. Keep your existing Vite environment variables and other project files.

## Supabase migration
Run `supabase/feature-update.sql` using Supabase SQL Editor. It adds `reservations.viewing_slot_id`, validates viewing-slot selection, and adds narrowly scoped admin permissions for photo metadata and Storage deletion, plus a viewings read policy. **Review conflicts with existing policies before running.** It does **not** add a new unrestricted administrator delete privilege for whole property records.

## Confirmation email
1. Create a Supabase Edge Function called `application-confirmation` using `supabase/functions/application-confirmation/index.ts`.
2. Set `RESEND_API_KEY` in Supabase Edge Function secrets. Ensure `SUPABASE_URL`, `SUPABASE_ANON_KEY`, and `SUPABASE_SERVICE_ROLE_KEY` are available as function secrets (Supabase may provide them automatically depending on setup). Do **not** put service role or Resend keys in Vite/Vercel public environment variables.
3. Verify the sender `no-reply@varsity-stay.com` in Resend. Deploy the function with JWT verification enabled.
4. Test a real signed-in student's submission and confirm an email arrives. The client warns if the application saves but the email fails. For guaranteed retries or exactly-once delivery, implement a database outbox plus a worker; this version sends upon client invocation.

## Test flows
- Landlord submits up to 15 photos (5 MB each), offers future viewing slots in dashboard.
- Student opens published property and navigates all images with arrows and thumbnails.
- Student selects a viewing request, chooses a future time, submits, sees review popup and receives email once the Edge Function is deployed and configured.
- Admin opens Property approvals, sees uploaded photos for pending listings, navigates them, and deletes an individual uploaded photo. Requires existing valid storage upload/read rules, and a public `property-images` bucket for image previews in the current architecture.
- Admin still uses existing approval/audit RPC to publish a verified property.

## Important privacy and security notes
Current original code stores public Storage URLs; an unapproved listing's **URL may still expose its photos** even though the listing record is hidden. Before launch, consider private buckets and signed read URLs (requires frontend changes). Validate your existing grants/RLS and viewing-slot schemas before running the migration. Admin photo deletion removes the storage object and then metadata; if the second operation fails, a metadata row may need manual cleanup.

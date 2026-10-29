# Changes we made to upstream code

When you merge a newer upstream release, check whether each of these still applies or upstream fixed it.
Each changed spot is marked `crm-desk patch`.

| File | What | Why |
|---|---|---|
| `desk/src/components/Customer360.vue` (new file) | Customer 360 card + dialog, data from `insurance.api.get_customer_360` | Insurance 360 view on tickets. **Identical copy** in the crm repo, `frontend/src/components/Customer360.vue`: change both together. |
| `desk/src/components/ticket-agent/TicketDetailsTab.vue` | Import + render `<Customer360>` above the Overview section, keyed on the ticket's `contact` / `raised_by` | Agents see the customer's policies, claims and premiums on the ticket. |
| `helpdesk/api/auth.py` (`get_current_user_email_info`) | Also returns `helpdesk_emails`: every Email Account address | Lets the composer know the helpdesk's own inboxes (incoming ones too). |
| `desk/src/components/ticket-agent/timeline/TicketTimeline.vue` (`isSelf`, `reply`, `replyAll`, `splitRecipients`) | "Own message" = sender is the user id, the user's email or a helpdesk inbox (was: user id only). Own addresses never become recipients; if To would be empty, Cc moves up to To, else the ticket's `raised_by`. Exclusion compares bare addresses (`"Name" <a@b.com>` = `a@b.com`). | Reply All put the support inbox into Cc; replying to your own email as Administrator (id `Administrator` ≠ `admin@example.com`) sent it to yourself with the customer in Cc. Upstream bugs, independent of the insurance app. (Was in `EmailArea.vue`, which upstream folded into `TicketTimeline.vue`; upstream fixed the Administrator part.) |
| `desk/src/components/CustomerDocumentsPicker.vue` (new file) | Attach picker listing the ticket customer's policy documents (`insurance.api.get_ticket_documents`), with "Upload from computer" | Agents can send a customer their policy documents from the reply composer. Warns when a recipient isn't one of the customer's emails. |
| `desk/src/components/EmailEditor.vue` | Paperclip opens `CustomerDocumentsPicker` when the customer has documents; otherwise the file dialog as before | Same feature. Attachments are copies (`insurance.api.attach_ticket_document`), since removing an attachment deletes its File. |
| `desk/src/components/BrandLogo.vue` | Logo `h-8 w-8 object-cover` → `h-8 w-auto object-contain` | Wide brand logos were cropped to a 32px square. Now shown whole, like CRM. |
| `desk/src/components/UserMenu.vue` | `<BrandLogo>` gets `max-w-16` (expanded) / `max-w-8` (collapsed sidebar) | Caps the now-natural logo width, same 64px limit CRM uses. |
| `desk/src/components/telephony/CallNote.vue` (new file) | Call-note panel: opens when a call is placed from a ticket, stays open after hang-up, saves an internal HD Ticket Comment (`new_comment`) with call context; agent text is HTML-escaped | Agents can record what was said on a call. Never emailed. |
| `desk/src/components/telephony/CallUI.vue` (`makeCall`, `makeCallUsing`) | Captures the ticket when an outgoing call starts and mounts `CallNote` | Notes attach to the ticket the call was placed from, not a stale `linkDoc`. Incoming calls (no ticket) get no note panel. |

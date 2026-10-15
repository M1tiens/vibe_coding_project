# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

Traditional marketplace dashboards overwhelm personal trainers with passive, descriptive reporting rather than actionable direction. Based on baseline research from "The Dashboard Nobody Reads," the existing experience suffered from acute friction:
- **60% bounce rate**, measured as trainer sessions lasting under 15 seconds with zero interaction.
- **12 disparate charts** cluttering the default landing view without clear priority.
- **6 clicks and 3 filters** required to locate the single most requested operational metric.

Direct qualitative feedback from trainers highlighted the paralysis:
> *"I open it, see twelve charts, and have no idea which one I'm supposed to act on. So I close it."* — Personal Trainer  
> *"The number I need is in there, but it takes six clicks and three filters to find it."* — Personal Trainer

In a two-sided fitness marketplace, trainer retention and revenue are directly tied to client session consistency. When clients go cold, trainers lose income and marketplace liquidity suffers. The core hypothesis tested by this prototype is:
> **Hypothesis:** Leading with a single headline metric (*clients at risk of lapsing*), a plain-language takeaway naming specific clients, and a one-tap recommended action (*send check-in and rebooking link*) will cause trainers to act immediately rather than bounce. An observable kill switch / pivot control verifies whether engagement translates to retained sessions or if the headline metric itself requires re-evaluation.

## Users & jobs

- **Primary user:** **Independent Personal Trainer (e.g., Alex Carter)**   A fitness professional managing 15–30 active 1-on-1 and small-group clients on the marketplace platform. They operate on mobile and desktop between sessions, have minimal administrative time, and depend on consistent client rebooking to maintain income.
- **Job to be done:** > **When I open my trainer workspace between sessions,**   > **I want to** immediately identify which clients are dropping off and see my daily operational schedule,   > **So that I can** re-engage lapsing clients with zero manual friction, keep my calendar booked, and avoid wasting time deciphering analytics reports.

## Scope

- **In:** 1. **Action-First Retention Overview (`/`):**
   - High-contrast focus band with headline count of at-risk clients (7+ days inactive).
   - One-tap bulk check-in action ("Send check-in to all 5") and individual modal message composition.
   - Individual client risk cards detailing inactivity days, primary fitness goal, and pre-composed SMS/chat template.
   - Simulated send feedback, optimistic state tracking, and batch confirmation toasts.
2. **Calendar Availability Management (`/calendar`):**
   - Weekly bookable hours total and active time-block distribution.
   - Time-block creation form (day of week, start time, end time with validation).
   - Interactive weekly list with block deletion to close booking windows.
   - Cross-navigation to daily bookings.
3. **Daily Bookings Operational View (`/bookings`):**
   - Four high-density summary metrics (active sessions, pending requests, total floor hours, first/last session span).
   - Chronological session cards displaying client avatar, session type, duration, and studio/online location.
   - Interactive session acceptance ("Accept"), cancellation ("Cancel"), and restoration ("Restore").
4. **Hypothesis & Experiment Tracker (`/experiment`):**
   - Documentation of baseline metrics (60% bounce, 12 charts, 6 clicks) and interview quotes.
   - Live click counter tracking check-in interactions across the prototype session.
   - Scenario previewer (`Pending`, `Promising`, `Pivot`) and observable experiment pause/pivot control.
5. **Global Shell & State Architecture:**
   - Unified sidebar navigation (`Overview`, `My calendar`, `Day bookings`, `The experiment`).
   - Persistent trainer profile badge displaying name, avatar, and rating (`★ 4.9 (128 reviews)`).
   - Standardized 1-second simulated network latency with skeleton screens across all primary views.
   - Interactive state switch (`Ready`, `Empty`, `Error`) with contextual empty states and inline retry error boundaries.
- **Out (explicitly):** 1. **Persistent Multi-Tenant Backend:** Database persistence (Supabase / PostgreSQL), user authentication, trainer profile editing, and role-based access control.
2. **Live Messaging Dispatch:** Real-world SMS (Twilio), WhatsApp Business API, push notifications, or email transport.
3. **External Calendar Synchronization:** Bidirectional sync with Google Calendar, Apple Calendar, or Outlook via CalDAV/OAuth.
4. **Marketplace Payment & Escrow:** Handling hourly rates, package deductions, cancellation fee penalties, or payout processing.
5. **Client-Facing Mobile Portal:** Client app views for receiving check-ins, picking time slots, or confirming sessions.

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | When viewing the dashboard overview | Must | display the exact count of clients at risk (>=7 days since last session with no forward booking), a plain-language summary naming key clients, and a primary CTA to send check-ins to all at-risk clients in one tap. |
| 2 | Clicking on an individual at-risk client card | Must | open a dialog pre-filled with a personalized check-in template including the client's first name, reassurance of past goals, and a booking link placeholder. Sending MUST mark the client card as contacted and increment experiment action metrics. |

## Data & events

_What gets stored, what gets tracked._

| Entity / State | Current Prototype Implementation (Mocked) | Required Production Implementation (Real Backend) |
| :--- | :--- | :--- |
| **Client Retention Record** | In-memory array (`src/lib/demo-api.ts`) with hardcoded client names, initials, inactive days (7–12), and goals. | `clients` table linked to `sessions` table. Calculated via SQL/view: `CURRENT_DATE - MAX(completed_at) >= 7` filtered where `upcoming_sessions_count = 0`. |
| **Check-in Dispatch** | React component state (`sent: string[]`); toast notification; increments in-memory experiment counter. | Async message queue (e.g. AWS SQS / Supabase edge function) delivering SMS/WhatsApp/Push with tracking token and unique rebooking link. |
| **Calendar Availability** | Local TanStack Query cache (`slotsQuery`) seeded with 7 weekly blocks; new slots generated with `crypto.randomUUID()`. Resets on browser refresh. | `trainer_availability_slots` table with `trainer_id`, `day_of_week` (0–6), `start_time` (TIME), `end_time` (TIME), and effective date ranges. |
| **Bookings & Session Status** | Local query cache (`bookingsQuery`) with 5 static entries (`pending`, `accepted`, `cancelled`). In-memory status updates. | `bookings` table with `id`, `client_id`, `trainer_id`, `scheduled_at`, `duration_minutes`, `status` (`pending`, `confirmed`, `cancelled`, `completed`), and audit log. |
| **Trainer Profile & Rating** | Hardcoded text and avatar asset in `AppShell` (`Alex Carter`, `4.9`, `128 reviews`). | `trainers` table with verified aggregated rating `AVG(rating)` and `COUNT(reviews)` computed from marketplace review submissions. |
| **Screen State Simulation** | Client URL search params (`?state=ready|empty|error`) with 1000ms `setTimeout` synthetic delay. | Production HTTP response codes (200, 404, 500) with TanStack Query error boundaries and network-level retries. |

## Open questions

1. **Lapse Threshold Definition:**
   - *MVP Rule:* Fixed 7-day inactivity threshold (no sessions completed in >=7 days and no upcoming session booked).
   - *Phase 2 Evolution:* Adaptive cadence multiplier (flagging at 1.5× the client's trailing 30-day average booking frequency).
2. **Delivery Channels for 1-Tap Check-ins:**
   - *Dispatch Policy:* Automatic dispatch through the client's saved preferred channel (SMS or WhatsApp), with fallback to in-app messaging. Trainers do not need to choose channels manually during 1-tap bulk sends.
3. **Rebooking Link Expiration & Concurrency Locks:**
   - *Collision Prevention:* Links open dynamic availability. When a client clicks an open slot, a temporary 10-minute hold is placed to prevent double-booking across concurrent client check-ins.
4. **Cancellation Policy & Notification Rules:**
   - *Workflow:* Any session status change (`accepted` or `cancelled`) automatically triggers instant push and SMS notifications to the client. A standard 12-hour cancellation window applies, prompting the trainer to waive or apply late-cancellation policies if cancelled within 12 hours.
5. **Experiment Validation & Pivot Criteria:**
   - *Kill-Switch Threshold:* Evaluated over a sample of 100 unique trainer sessions. If the bounce rate remains above 40% (vs. 60% baseline) or the primary check-in CTA click-through rate falls below 20%, the experiment state pivots to re-evaluate the leading metric.

### Future Open Questions
1. **Marketplace Calendar Integration:** Should availability integrate directly with Google Calendar/iCal or maintain the marketplace as the authoritative booking calendar?
2. **Client Rebook Conversions:** What secondary follow-up sequence should trigger if a client opens the rebooking link but does not complete a slot selection within 48 hours?

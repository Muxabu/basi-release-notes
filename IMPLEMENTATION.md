# Basi Implementation Guide

This guide explains what changed in the Family app, Driver app, backend, and browser pairing tool. It follows the data from the screen where an action starts to the modules that save it, enforce its rules, or update another screen.

[Family app](#family-app) · [Driver app](#driver-app) · [Backend](#backend) · [Browser pairing tool](#browser-pairing-tool)

## How the parts work together

The backend records the bus position and confirmed stops for the shared run. Each scheduled ride keeps its own pickup and drop-off assignment. The Driver app uses the route and counts at each stop. The Student and Parent views get the shared progress and their linked rider's assignment.

| Action | How it travels through Basi | Result |
| --- | --- | --- |
| The bus moves | The Driver app sends GPS fixes. The backend saves the latest location and publishes a realtime update to the driver and linked Family accounts. | The maps catch up to the current bus position. A snapshot reload restores the latest state after reconnecting. |
| The driver confirms a stop | The backend checks the assigned driver and stop order, updates the affected rides, records alerts for their linked accounts, and publishes the new run state. | Pickup moves the affected ride to in progress. Drop-off completes it. The final stop closes the run. |
| A student or parent changes a stop | The Family trip editor sends the requested assignment. The backend checks ride access and the remaining stop order, saves the change, and signals the affected views to refresh. | Both linked Family views show the new stop. The Driver stop list refreshes its counts. |
| A bus alert is tapped | The inbox or configured Apple push alert carries the scheduled ride identity. The Family app resolves it against the current session. | The correct trip opens from the alert. |
| The apps draw the route | Both apps ask for the same ordered street geometry separately from the live tracking snapshot. | The map shows street directions when available and can still display stops and the live marker during routing failures. |
| A rider pays for a card ride | Family resumes the stored checkout and submits the provider payment reference. The backend verifies it with Moyasar before dispatch. | Retries reuse the checkout and the backend advances an eligible booking once. |

## Family app

The Family app gives eligible students and linked parents access to the same student ride. Trip details connect their assignment, the driver's location, and alerts for their pickup and drop-off.

### Startup and screen repairs

| Screen or module | Implementation |
| --- | --- |
| App startup | Corrected the startup imports and wired in push registration, notification deduplication, and logout cleanup. |
| App configuration | Fixed the imports for loading configuration from the backend, mapping responses, and applying default legal links. |
| Home banners | Fixed the carousel repository's networking, response processing, and error imports. |
| Support chat | Fixed the support repository's network client, response mapping, and error imports. |
| Subscription list | Restored the theme, close button, and navigation imports. |
| Subscription detail | Restored the theme, loading state, close button, and navigation imports. |
| Subscription plans | Restored the theme and close button imports. |

### Scheduled ride details, map, and stop editor

**Trip details.** The screen opens with the selected ride and its own controller. The controller reads scheduled ride details and, for a shared run, adds the Bus tracking card. The card shows the next stop, the student's pickup and drop-off, and the moving bus marker. The detail screen also shows when a ride is in progress.

**Live data and reconnect.** The ride repository loads tracking and road geometry and saves updated stops. The tracking model keeps stops in route order and uses the newest valid location. The realtime service delivers bus updates to the controller that is following that run. On reconnect the controller reloads trip details and the latest snapshot. It also polls for fresh state while the ride is open and guards a saved stop assignment against a response that started before the edit.

**Street route and bus marker.** The map frames the ordered stops and the bends in the shared street route. The route module can use Google directions on a Google Maps view when available. Legs with missing road geometry are left open. Small GPS deviations can be aligned to the route for display, and a larger deviation stays at its reported position. The original yellow bus artwork is shared with the Driver app. The icon renderer caches bus images and adds a direction arrow when a heading is available. The Apple Maps adapter places the custom marker using its anchor and layer order.

**Change my trip stops.** An eligible student or linked parent can choose future pickup and drop-off stops on their own ride. The editor checks that pickup comes first and keeps a confirmed pickup fixed. The repository submits the change to the backend, and the detail controller applies the accepted assignment. The linked student and parent see the revised stop and the driver sees the updated counts.

### Paid ride request and Moyasar checkout

**Request and resume.** A card booking signals that the app supports server confirmation. The request screen opens checkout using the created ride. Both paid ride detail views offer **Continue card payment** for eligible pending bookings with an existing checkout reference.

**Payment screen.** Checkout loads the fare and persistent reference from the backend. It reuses that checkout reference when the rider reopens checkout or returns from 3-D Secure. After receiving the payment result, the Family app submits the provider's payment ID for server verification. Once the backend confirms the payment, the screen opens live tracking for the paid ride.

### Notifications inbox, home, and account settings

**Notifications inbox and navigation.** Pickup and drop-off alerts carry the matching scheduled ride. The same navigation handler serves the inbox and notification taps, checks for an active session, marks the inbox item read, and opens that ride directly. The ride-specific controller keeps the destination tied to the alert that opened it.

**Home and notification lifecycle.** Signed-in home starts realtime updates and connects notification processing to app resume, reconnect, and preference changes. Automatic reopening of a paid ride stays scoped to the active home screen. Inbox refreshes run one at a time. The local notification module uses a shared initialization and reports which alerts it presented. The inbox compares those alerts with deliveries already presented by iOS to suppress repeat banners.

**Preferences, eligible role switching, and sign-out.** The registration service ties the iOS push token to the current signed-in session and checks the Family notification preference and iOS permission. It refreshes registration after reconnection or token rotation. The existing Settings role switch removes the old registration before changing an eligible account's role and restores it if the switch fails. Sign-out also clears registration before revoking the session. The network client omits push-token traffic from debug logging.

### Native iOS notification bridge

The app delegate registers for Apple notifications, forwards token updates and bus alerts to Flutter, and clears previously presented arrival alerts on unregistration. The scene delegate forwards notification taps when iOS opens the app, and the native bridge can hold the tap until Flutter is ready. Debug, Profile, and simulator builds select the development Apple push environment. Device Release builds select the production environment. The runtime configuration and iOS entitlement both use the selected environment.

### Family regression tests

- Live map tests cover bus updates and reconnect, road routing and retries, map framing, and stop changes that overlap an earlier refresh.
- Road geometry tests cover stop order, failed or malformed geometry, and nearby GPS correction.
- Alert tests cover pickup and drop-off navigation from a cold start, separate controllers for separate rides, inbox recovery, and duplicate suppression.
- Registration tests cover consent and permission checks, token rotation, retry, and cleanup before a role change.

## Driver app

### Shared bus map and station counts

Opening a linked subscription ride brings up the shared bus map. The screen shows the street route, the moving bus marker, ordered stops, and arrival progress. The driver confirms the active stop directly on the map. The route panel shows its loading state and offers a retry when road geometry is unavailable.

The station sheet shows pickup and drop-off counts at each stop and highlights the next stop. It refreshes while open when a passenger changes an assignment. Station markers also show their pickup and drop-off counts.

### Tracking state and connection

The tracking model orders stops, records arrival progress, preserves the difference between a zero count and an unavailable count, and validates GPS coordinates and headings. The Driver API client fetches tracking state and road geometry separately. It publishes location fixes and confirms arrival at a specific stop.

The navigation controller refreshes the shared run on relevant realtime events and after reconnecting or returning to the app. It checks the run before applying updates and ignores snapshots that roll back arrival progress. The location queue retains the most recent fix and retries temporary failures while the shared map is active. The location service releases the old stream handle before cancellation so a restarted stream can keep its own handle. Startup preserves explicit backend settings when it fills gaps from the bundled sample.

### Street geometry and bus marker

The road loader prefers the shared geometry from the backend. On Google Maps it can request valid road legs from the configured Directions service. Unavailable legs appear as gaps, and the map fits the stops and available route into view. The displayed marker can correct small GPS drift near the route. Location uploads use the original validated fix.

The map adapter accepts a custom bitmap annotation on Apple Maps and allows the shared bus screen to disable rotation gestures. The icon renderer caches bus images and draws the direction indicator into the bitmap so the heading is visible on Apple Maps.

### Driver regression checks

The station sheet tests check updates at both affected stops while the sheet stays open, including zero and missing counts. Tracking tests cover stop order, arrival eligibility, GPS validation and publishing, directional icons, street route recovery, and location retry after the app resumes.

## Backend

### Shared-run provisioning and schedules

The administrative scheduling module groups existing subscription rides into a shared run. It checks that the selected rides are still scheduled, have active subscriptions, and share a service date, direction, and assigned driver approved for subscription work. Staff enter labeled stops in visit order and assign an earlier pickup and a later drop-off to each ride. The service creates the run, saves its stops and assignments together, and records an administrative audit event.

The subscription scheduler preserves those assignments when rebuilding generated rides. Trip summaries report the driver assigned to the run. The trip list and its counts show an active shared ride as in progress.

### Tracking, arrivals, and station changes

The Driver module lets the assigned driver read the run, submit GPS fixes, and confirm the next stop from the map. The Family module gives linked students and guardians a trip snapshot and lets them edit eligible stops for their assigned ride. The snapshot includes ordered stops, arrival times, assigned pickup and drop-off, and the latest bus position. The driver snapshot also includes scheduled pickup and drop-off counts. The access service resolves explicit subscription memberships and verified guardian phone matches inside the same database transaction when an update needs those checks.

The tracking service saves the latest accepted GPS fix and its heading when available. It publishes the shared position to the assigned driver and eligible Family accounts. When a driver confirms arrival, the service locks the run and checks stop order. A repeat confirmation returns the existing state. A new confirmation records the arrival, advances eligible pickups or completes eligible drop-offs, and closes the run at its final stop. The same transaction creates the relevant inbox alerts and any eligible push delivery records. After it commits, the realtime module sends the shared run update, individual ride changes, and each inbox alert to the corresponding channels. The tracking and inbox APIs give clients a way to reload the persisted state after a missed update.

A stop edit uses the same run lock as arrival confirmation, so the server evaluates eligibility against the latest recorded arrival. It checks the rider's access, active status, and the order of the selected stops. A confirmed pickup stays fixed, and a rider can still choose an eligible upcoming drop-off. The update refreshes linked Family views and the Driver stop counts.

### Street geometry

The maps module requests a driving route from the configured OSRM service using the stored stops in visit order. It validates the returned geometry against those stops, bounds network requests, and shares cached or concurrent work for matching coordinates. The authorized Driver and Family views read the same underlying route. They load street geometry separately from live trip data, so tracking and arrival confirmation continue if the router is unavailable.

### Card checkout and dispatch

The paid ride module gives new card bookings a persistent checkout reference so the Family app can resume the same attempt. After a 3-D Secure challenge, the return handler sends the rider back to checkout. The authenticated confirmation verifies the transaction with Moyasar using the server-calculated fare and currency, the provider's payment identifier and status, the refund state, and the ride and checkout details.

The database enforces a unique provider payment reference. The confirmation writes the paid state only while the booking is still eligible, and concurrent confirmation attempts can return the same successful state. The request that records the payment initiates the dispatch notification. Applicable staff assignment, forced completion, and trip advancement actions check that card payment has been confirmed. Driver operations filter active card ride lookups to eligible bookings.

### Account sessions and Apple push delivery

The account module registers and removes iOS push tokens for an authenticated session. Session issuance creates the refresh session before signing the access token, and authentication makes that session identity available to account handlers. Refreshed access tokens retain the binding. Logout removes the session's device registrations. A role change revokes the previous session, removes its push binding, and issues the replacement session in the same transaction.

Arrival processing creates separate pickup and drop-off inbox alerts. Each alert carries the trip reference needed to open the right ride. When APNs delivery is enabled, the notifications module queues a deduplicated record for each eligible device alongside the inbox entry. Before sending, the worker rechecks the account, role, session, device binding, and push preference. Workers use temporary leases to coordinate attempts, retry eligible temporary failures before expiry, and remove device tokens when Apple rejects them as invalid. The provider adapter authenticates to the configured Apple environment over HTTP/2.

### Schema, service startup, and coverage

The schema and three migrations add the checkout and verified payment references, shared runs with ordered stops and ride assignments, and session-bound device registrations with durable push delivery records. The service initializes the shared road router at startup, allows the app-identification header during browser CORS preflight, and owns the start and stop of the push worker. Configuration validation checks the Apple provider settings when delivery is enabled. The environment templates and backend guide document the provider settings, shared-run setup, and regression checks.

Backend integration tests exercise accepted and rejected payment confirmations, concurrent payment requests and reference reuse, authorized tracking, ordered arrivals, notification targeting, stop changes, and matching geometry across roles. The Apple push tests check request formatting, session registration and cleanup, retries, duplicate protection, and expired or ineligible deliveries with an injected sender. Unit coverage checks malformed road routes, stops missed by the route, redirect rejection, deduplication during router failures, and preservation of shared rides during schedule regeneration.

## Browser pairing tool

### Ride inspection and local lifecycle

The tool signs in an authorized Family account and Driver account through the configured backend. It scopes each browser session to its role and backend, finds the Family account’s active subscription in the Driver schedule, and compares the same ride’s identity, status, and driver phase through both APIs.

Remote backends support read-only ride inspection. On a loopback backend, a tester can confirm an optional exercise that starts a scheduled ride, records pickup, and completes it. The tool checks both views after each transition.

### Setup and regression checks

The browser guide covers local hosting, linked test accounts, and the conditions for inspection and the local exercise. Mocked checks cover matching a ride, all three local transitions, remote inspection, and the feedback shown when the driver is unlinked.

## Related guides

- Walk through the release milestones and see who receives alerts at each stop. [Change and Trip Flow](CHANGE_FLOW.md)
- Review the release summary. [Release Changes](RELEASE_CHANGES.md)
- Set up the backend and apps, then try the four-stop trip and station reassignment. [Run Guide](RUN_GUIDE.md)
- Review the completed checks and their results. [Testing](TESTING.md)

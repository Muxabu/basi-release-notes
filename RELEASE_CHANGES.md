# Basi Release Changes

Basi is a Saudi bus transport app for riders, families, and drivers. This release connects scheduled subscription rides to a shared bus route. The driver confirms each arrival from the map, while students and linked guardians can follow the bus and receive alerts for their assigned pickup and drop-off stops.

## App startup and browser pairing

The original Family app had 49 broken relative Dart imports across eight files, including its startup binding. The imports now resolve for app configuration, home banners, support chat, subscription screens, and startup dependencies. The Driver app fills missing configuration from its sample settings while preserving an explicitly configured API address and other values.

The browser pairing tool previously called retired scheduled-ride endpoints. It now uses the current Family and Driver APIs to compare the assigned ride, trip status, and driver phase. Its write controls operate on a local test backend, and remote use is limited to authenticated reads. The API also accepts the app-identification header in browser CORS preflight requests.

## Server-verified card payments

The original paid-ride flow could activate a card booking based on the confirmation received by the Family app. The current flow creates a persistent checkout reference so a rider can resume the same pending booking. The app submits the Moyasar payment reference to the backend, which independently confirms the paid status, refund state, amount, currency, and ride and checkout identifiers before it makes the booking available for dispatch.

The backend records the provider payment reference uniquely and handles repeated confirmations without dispatching the same booking twice. The relevant driver and staff operations also enforce confirmation before advancing a pending card ride. Riders completing a 3-D Secure challenge return to checkout and continue through the same authenticated server verification.

## Shared routes and live tracking

Subscription rides previously stored individual pickup and drop-off details. Staff can now group eligible scheduled rides into a shared run with an assigned driver, an ordered stop list, and a pickup and drop-off assignment for each ride. The server checks the subscription, service date, trip leg, driver eligibility, and stop order. Regenerating a subscription schedule preserves rides already linked to a run.

The driver can open the route within the scheduled trip and confirm the next stop on the map. The backend enforces stop order and handles a repeated confirmation safely. Confirming pickup advances the affected rides, and confirming drop-off completes the affected rides. The final stop closes the run.

The driver uploads GPS fixes to the shared run. Authorized students and linked guardians see the same bus position on their trip maps, and the trip screen labels an active ride as in progress. The apps receive trip-specific WebSocket updates and refresh from the current server state when they reconnect. The Family map also polls for updates, and the Driver app retries temporary GPS upload failures while retaining a newer queued fix when one is available.

## Rider stop selection

A student or linked guardian can change an eligible pickup or drop-off from the trip screen by choosing among the scheduled stops on their ride. The backend keeps pickup before drop-off and prevents changes to stops the bus has already reached. After pickup is confirmed, the rider can still change to an eligible upcoming drop-off.

An accepted change updates the shared assignment. The linked Family views refresh, and the driver sees the revised pickup and drop-off counts in the stop list while it remains open.

## Pickup and drop-off notifications

Each manual arrival creates the appropriate pickup or drop-off inbox alert for the student and linked guardian assigned to that stop. The alert opens the corresponding trip when tapped. The backend deduplicates repeated arrivals and delivery records, and the app coordinates inbox refreshes and push presentation to reduce duplicate alerts.

The Family app registers the iOS push token against the active account session and APNs environment. Logging out or switching roles clears the previous binding. The role switch in Settings is available to accounts enabled for role switching. With Apple push configured, a durable backend worker sends eligible alerts through APNs so iOS can present them when the Family app is closed. The worker checks the current session and notification preference before sending and can retry temporary delivery failures before the alert expires.

## Street-following maps and moving bus marker

The first shared tracking map joined stops with straight segments. The current map draws a driving route through the stops in order using OSRM geometry. Driver and Family views use the same route, and the Family map fits both the stops and the bends in the road. A yellow bus icon moves with GPS fixes and can show its heading.

The map can align a bus position up to 45 metres from the route to the road for display. A larger deviation remains visible at the reported position, and the original GPS fix is preserved on the server. If street geometry is unavailable, the map keeps the live position and stops visible. The Google Maps view can use Google Directions where configured, and legs without valid road geometry remain unconnected.

## Data and configuration for the changed modules

Three database migrations add verified payment references, shared runs and ordered stops with saved GPS positions, and session-bound iOS push tokens with durable delivery records. Station editing and street routing use those shared-run tables. Street geometry is loaded separately from live tracking so routing failures leave GPS and stop confirmation available. The backend validates and caches the route response so simultaneous views can share it.

Shared tracking uses WebSocket forwarding when the backend is behind a proxy. Shared street directions use **SCHEDULED_BUS_OSRM_BASE_URL**. Verified card checkout uses matching Moyasar settings and a reachable HTTPS 3-D Secure return. Arrival push delivery uses Apple provider authentication and the APNs environment that matches the signed Family app. [Run Guide](RUN_GUIDE.md)

## Tests performed

See what each completed test checked and its observed result. [Testing](TESTING.md)

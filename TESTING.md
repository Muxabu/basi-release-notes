# Basi Release Testing

These checks cover the changes to Basi's startup, shared bus trips, maps, arrival alerts, card checkout, and browser pairing. A rejection test passes when the system blocks the invalid action and leaves the affected state as expected.

[Startup and builds](#startup-and-build-checks) · [iOS Simulator](#ios-simulator-scenarios) · [Family app](#family-app) · [Driver app](#driver-app) · [Backend](#backend) · [Browser pairing](#browser-pairing-checks)

## Startup and build checks

| Check | What it tested | Result |
| --- | --- | --- |
| Family import scan | Rechecked local Dart imports after the startup, subscription, configuration, home banner, and support fixes. | **Passed.** The scan went from 49 unresolved imports across eight files to zero. |
| Flutter analysis | Analyzed the changed Flutter files in the Family and Driver apps. | **Passed.** The targeted analysis completed successfully. |
| iPhone Simulator startup | Built and launched both apps against the configured test backend. | **Passed.** The Family and Driver builds installed and opened. |

## iOS Simulator scenarios

### Live position, station changes, and roads

| Check | Result |
| --- | --- |
| Track one bus across Driver, Student, and Parent at the same time | Two successive GPS positions appeared on all three open maps. |
| Move the Student’s upcoming drop-off while the Driver’s station panel stays open | The Student saved the new stop and the linked Parent view showed it. The old stop’s drop-off count fell from one to zero and the new stop’s count rose from one to two in the open panel. |
| Replay a recorded street route and update the bus position | The Driver and Student maps displayed the same road line and the bus marker at both reported positions. API checks returned the same 172-point route through four ordered stops for Driver, Student, and Parent. A road-route request for an unrelated household’s ride was denied. |

### Pickup and drop-off alerts

The Driver confirmed four stops in order. Each stop created two notices for the Student and Parent linked to the affected ride.

| Stop | Event | Recipients |
| --- | --- | --- |
| 1 | Pickup | First linked Student and Parent |
| 2 | Pickup | Second linked Student and Parent |
| 3 | Drop-off | First linked Student and Parent |
| 4 | Drop-off | Second linked Student and Parent |

The first ride completed at stop 3. The second ride and the shared run completed at stop 4. Each assigned notice was injected in its own iOS Simulator pass with the Family app closed. All eight corresponding Home Screen banners appeared.

## Family app

All 24 checks below passed.

### Shared trip, stop selection, and map

| Check | Observed result |
| --- | --- |
| Shared trip snapshot | The app preserved the stop order, the passenger's pickup and drop-off, and the next stop. |
| Live updates and reconnect | The trip ignored updates for other runs, applied its own bus location, and refreshed stop state when an arrival came in or the connection returned. |
| Live bus and personal arrival | The bus icon changed position and heading, the road line stayed in place, and the map showed a drop-off arrival only when the passenger's own stop was reached. |
| Route ownership | The app accepted road geometry for the current run and discarded a response for another run. |
| Failed route fetch and retry | Stop markers stayed visible, the map showed that the road route was incomplete, and refresh retried the request. |
| Four stops on a short map | The camera framed all four stops, and the map displayed the bus icon alongside the stop markers. |
| Stop save during an older refresh | The new assignment stayed selected after the older tracking response arrived. |
| Station picker after pickup | The picker locked the completed pickup, offered a later drop-off, and saved the selection once. |
| Shared street geometry and GPS detour | The app used the supplied street route and its attribution. A small GPS deviation aligned to the road, and a wider detour kept its reported position. |
| Stop order and partial road geometry | The route builder linked stops in order, skipped duplicate coordinates, and retained the successful leg when a later leg failed. |
| Invalid road geometry | The malformed route produced no route line, and the map retained the reported position. |

### Pickup and drop-off notification links

| Check | Observed result |
| --- | --- |
| Open a pickup alert after a fresh launch on Student home | The alert opened the correct trip even when the trip was absent from the list and marked the inbox entry as read. |
| Open a drop-off alert after a fresh launch on Student home | The alert opened the correct trip even when the trip was absent from the list and marked the inbox entry as read. |
| Open the Parent inbox pickup alert | The alert opened the correct trip and marked the entry as read when the category was supplied on the inbox entry itself. |
| Open the Parent inbox drop-off alert | The alert opened the correct trip and marked the entry as read when the category was supplied on the inbox entry itself. |
| Open two trip details in succession | Each trip displayed its own details and kept its state separate. |

### iOS notification registration and inbox recovery

The registration checks used simulated iOS responses.

| Check | Observed result |
| --- | --- |
| App consent and iOS permission | The app registered after both permissions were granted and removed the registration when app consent was withdrawn. |
| Role switch during registration | The app finished the pending registration, removed it from the outgoing session, and attached it to the incoming session. |
| Registration failure and token rotation | The app retried a failed registration and removed the old token before registering the replacement. |
| Notification taps and foreground receipts | The app processed a queued launch tap after its listener was ready and handled subsequent taps and foreground receipts. |
| Background pickup and drop-off recovery | Both alerts were kept for recovery on resume and shown once. |
| Apple push and inbox fallback | Previously presented or opened alerts were suppressed in the inbox fallback, which still showed an arrival that had not been presented. |
| Overlapping inbox refreshes | A follow-up refresh picked up the arrival that came in during the first request and displayed it once. |
| Connection recovery and preferences | Inbox recovery honored the notification preference and fetched a missed arrival once the app reconnected with the preference enabled. |

## Driver app

All nine checks below passed.

| Check | Observed result |
| --- | --- |
| Stop order and next arrival action | The app displayed the shared stops in order, counted the reached stop, and offered the arrival action for the current stop. |
| Completed or already reached stop | The arrival action was unavailable when the current stop was already reached or the run was complete. |
| GPS input validation | Invalid coordinates were rejected, and an invalid direction was cleared while valid location data was retained. |
| Bus icon direction | The bus icon changed orientation independently of its GPS position. |
| Missing street geometry and retry | The map initially left the road line empty, then drew the returned street route and attribution after a later request succeeded. |
| Shared trip, location and arrival | The app loaded the shared trip, submitted the driver location and heading, and confirmed arrival at the intended stop. |
| GPS retry after app resume | A stationary location that failed to send was retried on resume. A newer fix received during the retry was sent next. |
| Missing passenger count versus zero | The trip preserved an explicitly reported zero and handled a missing count separately. |
| Stop counts in the open list | Moving a pickup updated the counts for both affected stops while the list stayed open. |

## Backend

All 24 checks below passed. Payment tests used simulated Moyasar responses. Apple push worker tests used an injected sender.

### Card payment verification

| Check | Observed result |
| --- | --- |
| Card checkout requirements | An updated client received a pending card checkout reference and the ride stayed out of dispatch. A client without support for this checkout was prompted to update. |
| Confirmation prerequisites | A missing reference, a mismatched reference, and a missing server payment configuration were each rejected without contacting Moyasar. The ride stayed pending. |
| Unknown payment | A payment reference the provider could not find was rejected, and the ride stayed pending with no recorded payment. |
| Pending payment | A payment still in progress at the provider left the ride pending with no recorded payment. |
| Declined payment | A failed payment left the ride pending with no recorded payment. |
| Refunded payment | A payment with a refund left the ride pending with no recorded payment. |
| Incorrect amount | A payment for a different amount left the ride pending with no recorded payment. |
| Incorrect currency | A payment in a different currency left the ride pending with no recorded payment. |
| Incorrect ride details | A payment whose metadata identified a different ride left the booking pending with no recorded payment. |
| Provider unavailable | A failed provider request left the ride pending with no recorded payment. |
| Successful confirmation and retries | Two concurrent confirmations returned the paid ride and saved the same payment reference. Later confirmation with that reference reused the stored result, and a different reference was rejected. |
| Payment reference reuse | A reference already recorded against another ride could not pay a second ride, which stayed pending. |

### Shared bus tracking and scheduling

| Check | Observed result |
| --- | --- |
| Shared trip and targeted arrivals | The backend published both driver GPS fixes to authorized subscribers and returned the saved location in the trip snapshot. An out-of-order arrival was rejected, repeat confirmations returned the same state, and the final stop completed the run. Each linked student and guardian received only the expected pickup and drop-off alerts, and unrelated accounts received no bus updates. |
| Rider stop changes | Linked riders could update their future stops. Both linked Family views and the Driver counts reflected the change. The test covered invalid order, access isolation, an edit racing arrival, a fixed confirmed pickup, and rejection after trip completion. Alerts followed the final assignments. |
| Shared street geometry | The Driver, Student, and Parent received identical ordered geometry from one provider request. Unauthorized or unlinked reads could not retrieve it. Disabling the router returned the run with empty geometry. |
| Schedule regeneration | Regeneration preserved a scheduled ride already assigned to a shared bus. |

### Road routing

| Check | Observed result |
| --- | --- |
| Invalid provider routes | Malformed responses and routes with invalid geometry, missing or reordered stops, or excessive stop offsets produced no road route. |
| Redirects and repeat failures | The router rejected redirects and invalid provider settings. Concurrent and immediate repeat requests for the unavailable route shared one provider attempt. |

### Apple push delivery logic

| Check | Observed result |
| --- | --- |
| Alert request format | The request used the Family app topic, alert priority and expiration, and stable notification and collapse identifiers. |
| Device registration and logout | Registration required an authenticated, eligible session and a valid device token. Bindings stayed separate by Apple environment. Token refresh retained registration access, and logout removed the affected binding. |
| Temporary failure and concurrent workers | The first send failure persisted a scheduled retry. Concurrent workers made only one retry attempt when it became due, and the successful attempt marked the delivery sent. |
| Pickup and drop-off alert payloads | Pickup and drop-off queued separate alerts with the correct wording and trip data for the intended device. Repeated queueing produced no duplicates. |
| Role change | Switching roles revoked the previous session and cleared its push binding while preserving another session's binding. The replacement session could register again. |
| Ineligible and invalid-device cleanup | The worker discarded expired alerts and deliveries whose device binding, account role, or notification preference had changed. A device the sender reported as unregistered was removed, and the inbox entries stayed available. |

## Browser pairing checks

All three regression checks passed with mocked API responses.

| Check | Result |
| --- | --- |
| Match an assigned ride and exercise its local lifecycle | The Family and Driver views agreed on the scheduled ride. Both views reported the expected status and driver phase after starting the trip, confirming pickup, and completing it. |
| Inspect a ride through a remote API | The tool displayed the matched ride, used read requests for ride inspection, and kept the lifecycle control disabled. |
| Inspect with an unlinked driver | The tool explained that the subscription lacked the Driver link and left the ride selection empty. |

## Related guides

- Follow the trip through Driver, Student, and Parent views. [Change and Trip Flow](CHANGE_FLOW.md)
- See how each tested feature is implemented. [Implementation Guide](IMPLEMENTATION.md)
- Set up and walk through the shared trip. [Run Guide](RUN_GUIDE.md)

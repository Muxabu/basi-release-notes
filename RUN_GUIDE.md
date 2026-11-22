# Basi Run Guide

Use an authorized Basi source checkout to start the backend, launch the Driver and Family apps, and walk through a shared subscription bus run. The examples use iPhone simulators and test accounts. The guide gives English names for controls that may appear in Arabic in the app.

## Requirements

- **Backend.** Node.js and npm. The backend declares Node.js 18 or newer. Use a maintained version compatible with the checkout. The local database setup uses PostgreSQL 16.
- **Mobile apps.** A Mac with Xcode, an installed iPhone Simulator runtime, CocoaPods, and a Flutter SDK that includes Dart 3.11 or a later Dart 3 version. Both apps target iOS 15 and later.
- **Test access.** An authorized operator, an approved driver with subscription rides enabled, and two student accounts with their respective linked parent accounts. Use separate signed-in simulator sessions to see Driver, Student, and Parent at the same time.

## Start the backend

1. Start PostgreSQL and create a dedicated development database. Use the backend environment sample as the starting point for local settings.
2. Set **DATABASE_URL** and load the required authentication and test sign-in settings from your authorized environment. Enable **WS_ENABLED** for shared trip updates. If you use a local environment override, check that the backend and Prisma migration task both point to the same database.
3. Install the backend's npm dependencies from its project directory. Run the backend package tasks **db:migrate** and **db:generate** to apply the database migrations and generate the Prisma client.
4. Start the backend using its **dev** package task. Open **/api/v1/health** on the backend address and check that the service reports healthy and WebSocket support enabled. Keep the database and backend running while you use the apps.

### Settings by feature

| Feature | Settings |
| --- | --- |
| Shared trip and live location | Point both apps at the same reachable **API_BASE_URL**. Include the **/api/v1** prefix. Enable WebSocket forwarding if the backend is behind a proxy. |
| Street directions | Set **SCHEDULED_BUS_OSRM_BASE_URL** to an approved OSRM driving service with coverage for the route. The maps can still show the stops and live bus position while street directions are unavailable. |
| iOS Home Screen alerts | Enable **APNS_ENABLED** and configure **APNS_ENVIRONMENT**, **APNS_TEAM_ID**, **APNS_KEY_ID**, and **APNS_PRIVATE_KEY_PATH** on the backend. Use a signed Family build with a matching push entitlement and allow notifications in the app and in iOS. |
| Verified card checkout | Configure matching **MOYASAR_SECRET_KEY** on the backend and **MOYASAR_PUBLISHABLE_KEY** in the Family app. |

## Launch the Driver and Family apps

1. Open **apps/driver** and **apps/family** as separate Flutter projects in your IDE. For each app, copy its environment sample to **assets/env/app.env** and set **API_BASE_URL**. To reach a backend on the same Mac from iPhone Simulator, use **127.0.0.1** and the port reported by the backend.
2. Resolve each app's Flutter dependencies. Install its iOS CocoaPods dependencies when required. Start the desired iPhone simulators, select a simulator for each app in the IDE, and run each Flutter project. Rebuild the app after changing its bundled environment settings.
3. Sign in with the authorized test accounts. Use one Driver session and separate Family sessions for the student and parent views. Open additional Family sessions for the second pair, or review their in-app inboxes after the run.

## Prepare a four-stop test run

1. Link each student to their parent through the subscription workflow. If you are setting up new subscriptions, have the operator approve them to generate scheduled rides. Choose two eligible scheduled rides from active subscriptions with the same service date and trip direction, assigned to the same subscription-enabled driver.
2. Use your authorized admin tooling to create a shared bus run through the backend's admin API. Enter four named stops with map positions in visit order. Assign Student A to pickup at stop 1 and drop-off at stop 3. Assign Student B to pickup at stop 2 and drop-off at stop 4. Use an upcoming service date visible in the Driver app.
3. In Driver, open **Subscription rides**, select one of the assigned rides, and open the **Bus trip map**. In each Family session, open **My Trips**, choose the correct student's scheduled ride, and find **Bus tracking** in the trip details. Use **All** in Family to find the ride as its status changes.

## Walk through the trip

1. Keep the Driver map open and allow location access while using the app. Use the Driver simulator's location controls to place the bus at two different points along the planned route. Check that the bus marker moves in the Driver, Student, and Parent views. With street directions configured, check that both apps draw the same route through the stops.
2. Before completing the run, reconnect a Family session after the Driver has reported another position. Check that the Family map catches up to the latest saved bus position and stop progress.
3. From the Driver map, open the pickup and drop-off stop list to see the initial passenger counts. Return to the map and use **Arrived at the stop** for the next displayed stop. Confirm each arrival in order and check its result before moving on. To check a Home Screen alert with Apple push configured, put a registered Family app in the background before confirming its assigned stop.

   | Driver confirms | Expected result |
   | --- | --- |
   | Stop 1 | Student A's ride moves to **In progress**. Student A and Parent A receive a pickup alert. |
   | Stop 2 | Student B's ride moves to **In progress**. Student B and Parent B receive a pickup alert. |
   | Stop 3 | Student A's ride completes. Student A and Parent A receive a drop-off alert. Student B's ride remains in progress. |
   | Stop 4 | Student B's ride and the bus run complete. Student B and Parent B receive a drop-off alert. |

4. Use the bell on the Family home screen to open the notification inbox. Check that each pair received the alerts for its own ride. Tap one of the alerts and confirm that it opens the assigned trip.

## Try changing a drop-off

1. Start a fresh run with the same four-stop assignments. Confirm stops 1 and 2 and leave stops 3 and 4 ahead of the bus.
2. In Student A's or Parent A's ride, open **Change my trip stops**. Choose stop 4 as the drop-off and save. The confirmed pickup stays fixed.
3. Check that both linked Family views show stop 4. The Driver stop list should show drop-offs at stop 3 changing from **1 to 0**, and at stop 4 from **1 to 2**.
4. Confirm stops 3 and 4 in order. Both riders remain in progress after stop 3. At stop 4, each student and linked parent receives the drop-off alert for their own ride.

See how the three roles share trip progress and what changed in the release. [Change and Trip Flow](CHANGE_FLOW.md) · [Release Changes](RELEASE_CHANGES.md)

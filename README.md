# Basi Shared Bus Release

Basi is a bus transport app in Saudi Arabia. This release connects individual subscription rides to a shared bus run. The driver reports arrivals from the route map, and the student and linked parent follow the same bus. Each rider keeps their own pickup, drop-off, and arrival alerts.

[See how the apps work together](#how-the-apps-work-together) · [Follow a four-stop trip](#a-four-stop-trip) · [See why the changes matter](#why-the-changes-matter)

## How the apps work together

Staff assign scheduled rides to an ordered route. The Driver app sends location updates, and the driver confirms each stop in order. The shared run keeps the bus position and stop progress in sync across Driver, Student, and Parent.

```mermaid
flowchart TB
    accTitle: A shared bus run across Driver, Student, and Parent
    accDescr: Staff assign stops and the driver publishes position and arrivals. The shared run updates the student and linked parent views.
    staff["Staff<br/>Schedule and assign stops"] --> run["Shared bus run<br/>Live position and stop progress"]
    driver["Driver app<br/>GPS and stop confirmations"] --> run
    run --> student["Student<br/>Live map and own arrival alerts"]
    run --> parent["Linked parent<br/>Live map and child's arrival alerts"]

    classDef operations fill:#e0f2fe,stroke:#0284c7,color:#0c4a6e
    classDef driverRole fill:#fef3c7,stroke:#b45309,color:#451a03
    classDef shared fill:#0f5148,stroke:#2dd4bf,color:#ffffff,stroke-width:2px
    classDef family fill:#ede9fe,stroke:#7c3aed,color:#2e1065
    class staff operations
    class driver driverRole
    class run shared
    class student,parent family
```

## Why the changes matter

| Area | What changed | Why it matters |
| --- | --- | --- |
| Shared tracking | Individual rides share one run. The apps reload trip progress after reconnecting. | Families can see where their bus is and which stop comes next. |
| Arrivals | The driver confirms each stop on the map. Basi creates an alert for each affected ride. | Students and linked parents know when it is time to board or get off. |
| iOS alerts | Added push registration, queued delivery, and alerts that open the assigned ride. | With Apple push configured, iOS can show alerts while the Family app is closed. |
| Stop selection | A student or linked parent can change an eligible upcoming stop. The driver's stop list updates. | The driver sees how many pickups and drop-offs to expect at each stop. |
| Street directions | Both maps draw the same street route and move a bus icon with GPS. | Riders can see the streets the bus is expected to take and track its progress. |

## A four-stop trip

The driver visits these four stations in order. **Student A and Parent A** get their pickup and drop-off alerts at stops 1 and 3. **Student B and Parent B** get theirs at stops 2 and 4.

```mermaid
flowchart LR
    accTitle: Four stops with distinct pickup and drop-off assignments
    accDescr: Student A boards at stop 1 and leaves at stop 3. Student B boards at stop 2 and leaves at stop 4. Each linked parent receives the same relevant arrival alerts as their student.
    pickupA["Stop 1<br/>Pickup A"] --> pickupB["Stop 2<br/>Pickup B"]
    pickupB --> dropoffA["Stop 3<br/>Drop-off A"]
    dropoffA --> dropoffB["Stop 4<br/>Drop-off B"]

    classDef pairA fill:#ccfbf1,stroke:#0f766e,color:#134e4a
    classDef pairB fill:#ede9fe,stroke:#7c3aed,color:#2e1065
    class pickupA,dropoffA pairA
    class pickupB,dropoffB pairB
```

A pickup moves that student's ride to **In progress**. A drop-off completes the student's ride. The bus run closes after the driver confirms the final stop.

## How a stop change reaches the driver

The Family app lets the student or linked parent pick an eligible stop on the existing route. Pickup must come before drop-off. Once pickup is confirmed, the rider can still change to an eligible upcoming drop-off.

```mermaid
flowchart TB
    accTitle: A rider's stop change refreshes the Driver, Student, and Parent views
    accDescr: The service checks stop eligibility, saves valid changes to the ride, and updates both the Family views and the driver stop list.
    choose["Student or parent<br/>Select a new stop"] --> check{"Is the stop eligible?"}
    check -->|Yes| save["Update the ride assignment"]
    check -->|No| keep["Keep the current stop"]
    save --> family["Student and parent<br/>See the revised stop"]
    save --> driver["Driver<br/>Sees revised passenger counts"]

    classDef updated fill:#0f5148,stroke:#2dd4bf,color:#ffffff,stroke-width:2px
    classDef familyRole fill:#ede9fe,stroke:#7c3aed,color:#2e1065
    classDef driverRole fill:#fef3c7,stroke:#b45309,color:#451a03
    class save updated
    class choose,family familyRole
    class driver driverRole
```

If Student A moves their drop-off from stop 3 to stop 4, the count falls by one at stop 3 and rises by one at stop 4. The bus still visits the stops in the scheduled order.

## More in this release

- **Startup.** Repaired the Family app imports and preserved explicit Driver settings while filling missing defaults.
- **Card checkout.** Added server-side Moyasar payment verification and retry protection before dispatch. Riders can resume checkout after a 3-D Secure challenge.
- **Browser pairing.** Updated the tool to use the current scheduled-ride APIs for Driver and Family.

## Release documentation

- See how the project evolved and follow the ride from scheduling to the final stop. [Change and Trip Flow](CHANGE_FLOW.md)
- Review the release changes and their configuration settings. [Release Changes](RELEASE_CHANGES.md)
- See what changed in each screen and module and how the apps, backend, and providers work together. [Implementation Guide](IMPLEMENTATION.md)
- Set up the backend and both apps, then try a four-stop ride as Driver, Student, and Parent. [Run Guide](RUN_GUIDE.md)
- See what each completed check covered and its result. [Testing](TESTING.md)

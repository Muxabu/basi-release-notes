# Basi Change and Trip Flow

Basi connects a driver's scheduled bus run with each assigned student's ride and the linked parent's view. All three roles follow the same run. Each student keeps an individual pickup and drop-off assignment.

## How the project evolved

| Stage | What changed |
| --- | --- |
| Initial fixes | Corrected Family startup imports and preserved Driver configuration when filling missing defaults. Added server-side Moyasar confirmation before card ride dispatch and updated browser pairing to use current scheduled rides. |
| Shared bus runs | Grouped individual scheduled rides onto one ordered stop list for an assigned driver. Each ride gained its own pickup and drop-off on that list. |
| Tracking reliability | Added live driver position and stop updates, fresh state after reconnect, and retries for temporary GPS upload failures. Updated trip status and inbox recovery. |
| iOS notifications | Added session-scoped push registration, delivery queuing, and native handling that opens the correct ride when a bus alert is tapped. |
| Pickup and drop-off | Added distinct alerts for both arrival types and scoped them to the student and linked parent whose ride reaches that stop. |
| Station changes | Let the student and linked parent change eligible stops on their ride. The driver's stop counts and the linked accounts' assignments update after saving. |
| Street routing | Added shared driving geometry through the ordered stops. The maps display a yellow bus icon that moves with GPS updates and keep the stop markers visible if the street route is unavailable. |

## How a shared trip works

1. An operator creates an ordered run from existing scheduled rides and assigns each student a pickup and a later drop-off. The driver sees the route and the number of pickups and drop-offs at each stop.
2. The student and linked parent open the same ride assignment. Their maps show the bus location, the next stop, and the student's own pickup and drop-off. Where street directions are available, both apps display the same driving geometry.
3. The Driver app sends live GPS positions. The driver confirms **Arrived** at the next stop. The service advances the route in order and updates rides assigned to that stop. Repeating a confirmation leaves the existing arrival and alert records intact.
4. An eligible pickup advances the student's ride to in progress. A confirmed drop-off completes that ride. The final stop closes the bus run. Each arrival alert opens the corresponding ride in the Family app.

### Example with four stops

Student A boards at stop 1 and leaves at stop 3. Student B boards at stop 2 and leaves at stop 4. Each has one linked parent in this example.

| Driver confirms arrival | Ride progress | Arrival alert recipients |
| --- | --- | --- |
| Stop 1 | Student A's ride starts. | Student A and Parent A receive a pickup alert. |
| Stop 2 | Student B's ride starts. | Student B and Parent B receive a pickup alert. |
| Stop 3 | Student A's ride completes. | Student A and Parent A receive a drop-off alert. |
| Stop 4 | Student B's ride and the bus run complete. | Student B and Parent B receive a drop-off alert. |

Linked Student and Parent accounts can follow the common map and stop progress throughout the run. Personal arrival alerts are scoped to the affected ride.

## Changing a pickup or drop-off

The student or linked parent can use **Change my trip stops** on their own eligible ride. The picker uses existing stops on that run. Pickup must precede drop-off, and changes can use stops the bus has yet to reach. Once the driver confirms pickup, that pickup stays fixed. An eligible future drop-off can still be changed.

For example, Student A can move their drop-off from stop 3 to stop 4 before either stop is reached. Parent A sees the revised assignment. The driver sees stop 3's drop-off count decrease by one and stop 4's increase by one. The physical stop order stays the same. When the driver confirms stop 4, both students and their respective linked parents receive the drop-off alerts for their own rides.

## Reconnect and alert delivery

After reconnecting, the apps reload the latest run state, including the last recorded location, stop progress, and current assignment. The Driver app retries temporary GPS upload failures. The Family app refreshes its inbox and suppresses a repeat banner for an alert already shown.

Arrival alerts appear in the linked accounts' inboxes. On iOS, the Family app can register for Apple push when notifications are enabled. Tapping a bus alert opens the assigned ride.

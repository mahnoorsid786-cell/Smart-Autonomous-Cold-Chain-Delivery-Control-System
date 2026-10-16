Task 4: Operation Identification and Operation Schema

An operation is an action performed by the system. For example, CorrectTemperature() is an operation, whereas CONSERVING is a state.

4.1 Identification of 12+ operations

| Operation ID | Operation name              | Purpose                                                                                        |
| ------------ | --------------------------- | ---------------------------------------------------------------------------------------------- |
| OP-01        | ReceiveDeliveryRequest      | Receives and validates a delivery request.                                                     |
| OP-02        | AuthenticateOperator        | Checks operator identity and role permissions.                                                 |
| OP-03        | PlanAndStartRoute           | Selects a route and starts navigation.                                                         |
| OP-04        | DetectAndHandleObstacle     | Suspends navigation and initiates obstacle avoidance.                                          |
| OP-05        | AuthorisePackageDelivery    | Checks destination and safety conditions before delivery.                                      |
| OP-06        | InitiateWarehouseReturn     | Abandons delivery or returns after completion.                                                 |
| OP-07        | EvaluateCriticalConditions  | Evaluates battery and temperature thresholds and triggers Safe Mode.                           |
| OP-08        | InitialiseChamberProfile    | Performs sensor self-check and loads batch limits.                                             |
| OP-09        | CorrectAndVerifyEnvironment | Corrects temperature/humidity and verifies recovery.                                           |
| OP-10        | RecordAuditEvent            | Records commands, faults, alarms, and state transitions.                                       |
| OP-11        | AuthoriseRailwayCrossing    | Checks barrier status and train detection before crossing.                                     |
| OP-12        | HandleCommunicationLoss     | Detects link loss and initiates a safe response.                                               |
| OP-13        | ManagePowerFailure          | Switches to emergency power or shuts down safely.                                              |
| OP-14        | ManageDoorInterlock         | Suspends conservation activities when the door opens.                                          |
| OP-15        | VerifyBatchRemoval          | Allows removal only when safe conditions are verified.                                         |
| OP-16        | StabiliseAfterVibration     | Suspends risky activities until vibration remains below the threshold for the required period. |
| OP-17        | ActivateProtectionResponse  | Reduces light exposure and alerts the operator when recovery fails.                            |
| OP-18        | DisplayRobotAndBatchRecords | Displays authorised robot, operator, and project records in the web portal.                    |

4.2 Complete operation schemas

OP-02: AuthenticateOperator

- Description: Validates the operator's identity and permissions before a remote command is accepted.
- Inputs: Operator credentials/token, requested command, robot ID, role.
- Outputs: Authentication result, authorisation result, rejection reason.
- Reads: Operator account, role permissions, token validity, robot access rules.
- Changes: Command status changes to ACCEPTED or REJECTED; no robot action is started before approval.
- Sends/Alerts: Sends an approval or rejection acknowledgement and records the decision.
- Assumes (precondition): A command has been received and the relevant identity information is available.
- Result (postcondition): The command is accepted only if authentication and role authorisation both succeed; otherwise it is rejected.
  
OP-04: DetectAndHandleObstacle

- Description: Detects an obstacle and coordinates safe avoidance.
- Inputs: Obstacle sensor readings, current route, robot position.
- Outputs: Obstacle status, alternative path or stop instruction.
- Reads: Navigation state, obstacle sensors, route map.
- Changes: Robot transitions from NAVIGATING to AVOIDING_OBSTACLE; it returns to NAVIGATING only after clearance is verified.
- Sends/Alerts: Updates Mission Control and logs obstacle detection and clearance.
- Assumes (precondition): Robot is navigating and obstacle sensors are operational.
- Result (postcondition): Navigation remains suspended until the route is confirmed safe; delivery cannot start directly from obstacle avoidance.
  
OP-07: EvaluateCriticalConditions

- Description: Checks whether battery or thermal values have reached critical thresholds.
- Inputs: Battery capacity, battery temperature, chamber temperature, threshold values, timestamps.
- Outputs: Critical-condition flag, Safe Mode trigger.
- Reads: Battery monitor, chamber sensors, configured thresholds, current robot state.
- Changes: Initiates SAFE_MODE when a critical condition is detected.
- Sends/Alerts: Sends a critical alarm to Mission Control and records the trigger and response timestamps.
- Assumes (precondition): Critical-condition monitoring is active and readings are sufficiently trustworthy.
- Result (postcondition): The robot enters SAFE_MODE no later than 3 seconds after detection of a specified CR-01 trigger.
  
OP-08: InitialiseChamberProfile

- Description: Performs chamber self-checks and loads the medicine batch profile.
- Inputs: Batch ID, temperature limits, humidity limits, sensor readings, door status.
- Outputs: Self-check result and profile-load result.
- Reads: Sensor health, batch database, chamber door sensor, required environmental limits.
- Changes: Enables chamber monitoring after a successful self-check and permits conservation only when the profile is loaded and the door is closed.
- Sends/Alerts: Reports sensor failures, unknown batch IDs, or missing profile information.
- Assumes (precondition): The chamber is powered and the batch information is available.
- Result (postcondition): Conservation remains disabled unless all essential sensors work, the profile is loaded, and the door is closed.
  
OP-09: CorrectAndVerifyEnvironment

- Description: Corrects temperature or humidity that is outside the batch's permitted range and verifies actual recovery.
- Inputs: Current temperature, current humidity, target limits, correction controls, recovery timeout.
- Outputs: Verified recovery status or correction-failure status.
- Reads: Live sensor readings, batch profile, elapsed recovery time, sensor health.
- Changes: Initiates environmental correction; transitions to normal conservation only after readings return to the allowed range. On failure, initiates protection mode.
- Sends/Alerts: Reports recovery success or sends a protection alarm to the operator.
- Assumes (precondition): A valid batch profile is loaded and relevant sensors are operational.
- Result (postcondition): Recovery is confirmed only by actual readings; failed recovery within the permitted period causes protection mode, reduced light exposure, and an alert.
- 
OP-14: ManageDoorInterlock

- Description: Suspends normal chamber operation when the door opens during conservation.
- Inputs: Door sensor status, chamber readings, active batch profile.
- Outputs: Door alarm, interlock status.
- Reads: Door sensor, temperature/humidity readings, sensor health, conservation state.
- Changes: Changes chamber state from CONSERVING to a suspended/protection state; normal conservation cannot resume until the door is closed and conditions and sensors are verified.
- Sends/Alerts: Logs door opening and alerts the operator if required by the safety policy.
- Assumes (precondition): Door monitoring is active.
- Result (postcondition): The chamber cannot report normal conservation while the door is open or required verification remains incomplete.
  
OP-15: VerifyBatchRemoval

- Description: Checks whether a medicine batch can safely be removed.
- Inputs: Removal request, door status, environmental readings, protection status, sensor health.
- Outputs: Removal permission or rejection.
- Reads: Chamber safety status, current readings, protection-response status, batch identity.
- Changes: Grants or denies removal permission; does not authorise removal while a protection response is active.
- Sends/Alerts: Logs the decision and informs the operator.
- Assumes (precondition): The batch and removal request can be identified.
- Result (postcondition): Removal is permitted only when the chamber is safe and no protection response is active.
  
4.3 Operations that change states

Example 1: Door opens during conservation
CONSERVING
OP-14: Door opens
SUSPENDED / PROTECTION
Door closed + sensors and conditions verified
CONSERVING
Related robot-model link: if the door opens while the robot is preparing to deliver, the delivery interlock must prevent DELIVERING until safe conditions are re-established.

Example 2: Critical temperature during navigation
NAVIGATING
OP-07: Critical temperature detected
SAFE_MODE
The transition is required by FR-07 and is verified by TC-06. Entry must occur within 3 seconds of detecting the CR-01 trigger.

4.4 Operations requiring sensor verification
1. OP-09 — CorrectAndVerifyEnvironment
Issuing a cooling or humidity-correction command does not prove the environment has recovered. A faulty actuator, open door, or continuing heat source may prevent recovery. The system must check actual sensor readings and confirm that values remain within the batch's permitted limits.
2. OP-15 — VerifyBatchRemoval
A removal command or an operator's claim that the chamber is safe is insufficient. The system must verify current environmental readings, sensor health, door status, and that no protection response is active before permitting removal.
The same principle applies to railway crossing clearance: a command to open a barrier is not evidence that the crossing is safe.

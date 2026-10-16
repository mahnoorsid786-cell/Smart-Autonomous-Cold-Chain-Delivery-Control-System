Task 5: Constraints, Formalisation and Violations — Railway Crossing

5.1 Ten-plus system constraints
A constraint defines a condition that must always hold during operation.

| Constraint ID | Constraint in simple English                                                                                                             | Why necessary                                                          |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| CON-01        | The robot shall navigate only after a valid delivery request is accepted.                                                                | Prevents unauthorised movement.                                        |
| CON-02        | Every accepted remote command must pass authentication and role authorisation.                                                           | Prevents unauthorised control.                                         |
| CON-03        | The robot shall not deliver while handling an obstacle.                                                                                  | Prevents unsafe delivery transitions.                                  |
| CON-04        | Delivery shall begin only after the destination is reached and delivery conditions are verified.                                         | Ensures delivery occurs at the correct location under safe conditions. |
| CON-05        | The robot shall not deliver while the chamber door is open or the chamber is unsafe.                                                     | Protects medicine quality.                                             |
| CON-06        | A critical battery condition must cause delivery abandonment and a defined safe response; normal delivery cannot continue.               | Prevents battery depletion during delivery.                            |
| CON-07        | A CR-01 trigger must cause entry into SAFE_MODE within 3 seconds.                                                                        | Limits the response time to critical faults.                           |
| CON-08        | Conservation shall not start unless essential sensors work, the profile is loaded, and the door is closed.                               | Prevents unsafe storage conditions.                                    |
| CON-09        | Temperature or humidity recovery shall be confirmed by actual sensor readings; failed recovery must activate protection mode.            | Avoids false recovery declarations.                                    |
| CON-10        | Safety events, accepted/rejected commands, and state changes must be timestamped and logged.                                             | Supports traceability and investigation.                               |
| CON-11        | The robot shall not enter a railway crossing unless the barrier is confirmed open and no train is approaching or present.                | Prevents collision with a train.                                       |
| CON-12        | Sensor failure or communication loss must prevent reliance on unverified crossing-clear or remote-command information.                   | Prevents unsafe decisions from missing or stale data.                  |
| CON-13        | The pilot shall support at least 20 simultaneously connected robots.                                                                     | Meets the pilot capacity requirement.                                  |
| CON-14        | Significant vibration shall suspend risky activities until vibration remains below the permitted threshold for the stabilisation period. | Prevents physical damage to sensitive medicines.                       |
| CON-15        | A batch may be removed only when the chamber is safe and no protection response is active.                                               | Prevents unsafe handling of medicines.                                 |
| CON-16        | Railway barriers shall remain closed while a train is passing and shall open only after crossing-clear status is confirmed.              | Protects both trains and road/crossing users.                          |

5.2 Formalisation of constraints using logical notation

Let the following Boolean variables represent observed system conditions:
- Train_Present: a train is present at the crossing.
- Train_Approaching: a train is approaching.
- Barrier_Open: the barrier is open.
- Crossing_Clear: the crossing is confirmed clear.
- Robot_Entering: the robot is entering the crossing.
- Authenticated: operator authentication succeeded.
- Role_Authorised: operator has permission for the command.
- Door_Open: chamber door is open.
- Chamber_Safe: chamber conditions are verified safe.
- Delivering: robot is in the delivery state.
- Obstacle_Active: an obstacle is being handled.
- Conservation_Active: chamber conservation is active.
- Sensors_Healthy: essential sensors are working.
- Critical_Condition: a CR-01 critical trigger has been detected.
- Safe_Mode: robot is in Safe Mode.
- Link_Lost: communication failure is detected.
- Protection_Active: chamber protection mode is active.

 | Constraint ID | Formal expression                                                                        | Meaning                                                                                            |
| ------------- | ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| CON-02        | `Command_Accepted → (Authenticated ∧ Role_Authorised)`                                   | A command is accepted only after authentication and role authorisation.                            |
| CON-03        | `Delivering → ¬Obstacle_Active`                                                          | Delivery and active obstacle handling cannot occur together.                                       |
| CON-05        | `Delivering → (¬Door_Open ∧ Chamber_Safe)`                                               | Delivery requires a closed door and verified safe chamber conditions.                              |
| CON-07        | `Critical_Condition → (Safe_Mode within 3 seconds)`                                      | A critical trigger must lead to Safe Mode within 3 seconds.                                        |
| CON-08        | `Conservation_Active → (Sensors_Healthy ∧ Profile_Loaded ∧ ¬Door_Open)`                  | Conservation requires healthy sensors, a loaded profile, and a closed door.                        |
| CON-09        | `Correction_Complete → In_Range_Verified`                                                | Correction is not considered successful until sensor readings confirm the values are within range. |
| CON-11        | `Robot_Entering → (Barrier_Open ∧ ¬Train_Present ∧ ¬Train_Approaching ∧ Crossing_Clear)` | Entry requires an open barrier, no train present or approaching, and confirmed clearance.          |
| CON-12        | `(Sensor_Failure ∨ Link_Lost) → ¬Unverified_Crossing_Entry`                              | Sensor failure or link loss must prevent entry based on unverified crossing information.           |
| CON-14        | `Significant_Vibration → ¬Risky_Activity`                                                | Risky activities are suspended during significant vibration.                                       |
| CON-15        | `Batch_Removal_Authorised → (Chamber_Safe ∧ ¬Protection_Active)`                         | Batch removal requires safe conditions and no active protection response.                          |
| CON-16        | `Train_Present → ¬Barrier_Open`                                                          | The barrier must not be open while a train is present.                                             |
| CON-10        | `Safety_Event → Event_Logged`                                                            | Every safety event must be recorded.                                                               |                                                        |

5.3 Violation scenarios, detection and expected responses

| Test ID | Constraint | Violation scenario and variable values                                                                              | Detection method                                                 | Expected response                                                                                                            |
| ------- | ---------- | ------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| TC-13   | CON-02     | `Command_Accepted=TRUE`, `Authenticated=FALSE`, `Role_Authorised=TRUE`                                              | Command-gateway access-control monitor                           | Reject command, alert/log the security violation, and prevent robot action.                                                  |
| TC-14   | CON-03     | `Delivering=TRUE`, `Obstacle_Active=TRUE`                                                                           | Robot-state invariant monitor                                    | Inhibit delivery, stop unsafe activity, log the violation, and initiate the defined safe response.                           |
| TC-15   | CON-05     | `Delivering=TRUE`, `Door_Open=TRUE`, `Chamber_Safe=FALSE`                                                           | Door interlock and chamber sensor monitor                        | Suspend delivery, raise an alert, log the event, and require verified safe conditions before resuming.                       |
| TC-16   | CON-07     | `Critical_Condition=TRUE`; detection at 10.0 s; `Safe_Mode=FALSE` at 13.1 s                                         | Timestamp-based safety monitor                                   | Flag the missed 3-second deadline, initiate the emergency response, alert Mission Control, and log the fault.                |
| TC-17   | CON-08     | `Conservation_Active=TRUE`, `Sensors_Healthy=FALSE`, `Profile_Loaded=TRUE`, `Door_Open=FALSE`                       | Chamber self-check and state invariant monitor                   | Suspend conservation, enter the appropriate protection state, alert the operator, and log the failure.                       |
| TC-18   | CON-09     | `Correction_Complete=TRUE`, `In_Range_Verified=FALSE`                                                               | Compare correction status against fresh sensor readings          | Reject the recovery claim, continue permitted recovery attempts, or enter protection mode after timeout.                     |
| TC-19   | CON-11     | `Robot_Entering=TRUE`, `Barrier_Open=TRUE`, `Train_Present=TRUE`, `Train_Approaching=FALSE`, `Crossing_Clear=FALSE` | Crossing safety monitor using train detection and barrier status | Prevent or stop entry if still safely possible, activate crossing alarms, alert control, and log the incident.               |
| TC-20   | CON-12     | `Link_Lost=TRUE`, `Sensor_Failure=FALSE`, `Unverified_Crossing_Entry=TRUE`                                          | Communication watchdog and crossing-entry interlock              | Block crossing entry, stop at a safe location, notify Mission Control when possible, and log link loss.                      |
| TC-21   | CON-14     | `Significant_Vibration=TRUE`, `Risky_Activity=TRUE`                                                                 | Vibration sensor and activity monitor                            | Suspend risky activities, alert the operator, and resume only after the stabilisation period is satisfied.                   |
| TC-22   | CON-15     | `Batch_Removal_Authorised=TRUE`, `Chamber_Safe=TRUE`, `Protection_Active=TRUE`                                      | Batch-removal interlock                                          | Reject removal, keep the protection response active, alert the operator, and record the attempt.                             |
| TC-23   | CON-16     | `Train_Present=TRUE`, `Barrier_Open=TRUE`                                                                           | Independent crossing safety monitor                              | Raise a critical alarm, command closure if safe and physically possible, alert railway control, and log the barrier failure. |
| TC-24   | CON-10     | `Safety_Event=TRUE`, `Event_Logged=FALSE`                                                                           | Audit-log completeness monitor                                   | Flag a logging failure, alert monitoring personnel, and use a protected local event buffer if available.                     |

5.4 How the system detects and responds to violations

The system should use multiple verification mechanisms rather than relying on a single software check.

1. Runtime safety monitoring
Continuously evaluates conditions such as train presence, barrier position, chamber temperature, door status, and robot state. When an invariant fails, it initiates the defined safety response.
2. Timed verification
Measures deadlines, especially the 3-second Safe Mode requirement. A late response is a test failure even if Safe Mode is eventually reached.
3. Timed verification
Measures deadlines, especially the 3-second Safe Mode requirement. A late response is a test failure even if Safe Mode is eventually reached.
4. Sensor and interlock verification
Compares commands with actual feedback. A barrier-open command does not prove the barrier is open; a cooling command does not prove the temperature is safe.
5. Audit and incident verification
Confirms that the triggering event, observed readings, transition, command decision, alarm, and recovery outcome have been logged with timestamps.

5.5 Final quality verification checklist

Use this checklist to review the completed assignment.

Assignment review

0 of 11

10 functional requirements and 5 measurable NFRs documented

4 high-risk requirements identified

Three change requests recorded with old and new wording

Multi-layer RTM connects requirements, states, operations, constraints, tests, and risks

At least 8 traceable scenario test cases and CR regression tests included

State definitions and transition table include valid, invalid, and missing transitions

Safe Mode and Link Lost behaviour defined, including recovery conditions

At least 12 operations identified and 6 complete operation schemas documented

Two sensor-verified operation postconditions explained

At least 10 constraints and 8+ logical formalizations included

At least 8 violation scenarios include detection and expected responses


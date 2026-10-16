Task 2: Change Requests and Traceability Matrix (RTM)

2.1 Version-controlled change log

| CR ID | Req. ID | Old requirement (V1.0)                                                                    | New requirement (V1.1)                                                                                                                                                   | Reason                                                    |
| ----- | ------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------- |
| CR-01 | FR-07   | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical threshold or battery capacity falls below the emergency level. | Adds measurable timing and explicit trigger conditions.   |
| CR-02 | NFR-03  | The system shall support communication with multiple robots simultaneously.               | The system shall support at least 20 simultaneously connected robots.                                                                                                    | Defines the minimum number of concurrent connections.     |
| CR-03 | FR-02   | Only authenticated operators shall be permitted to issue robot commands.                  | The system shall require authenticated and role-authorised operators before accepting any robot command.                                                                 | Adds explicit role-based authorisation to authentication. |   

2.2 Why the changed wording is more testable
- CR-01: A tester can inject a critical condition, record its detection time and Safe Mode entry time, and verify that the difference is no more than 3 seconds.
- CR-02: A tester can connect 20 robots concurrently and verify that all 20 connections are supported.
- CR-03: A tester can use valid and invalid accounts, as well as accounts with insufficient privileges, and verify that only authenticated, authorised commands are accepted.

 2.3 Multi-layer RTM

| Req. ID | State/transition                                             | Operation | Constraint     | Test case | Risk     |
| ------- | ------------------------------------------------------------ | --------- | -------------- | --------- | -------- |
| FR-01   | ST-01: IDLE → NAVIGATING                                     | OP-01     | CON-01         | TC-01     | High     |
| FR-02   | Authentication gate before command acceptance                | OP-02     | CON-02         | TC-02     | Critical |
| FR-03   | ST-01: IDLE → NAVIGATING                                     | OP-03     | CON-01         | TC-01     | High     |
| FR-04   | ST-02: NAVIGATING ↔ AVOIDING_OBSTACLE                        | OP-04     | CON-03         | TC-03     | Critical |
| FR-05   | ST-03: NAVIGATING → DELIVERING                               | OP-05     | CON-04, CON-05 | TC-04     | Critical |
| FR-06   | ST-04: DELIVERING → RETURNING; battery emergency → RETURNING | OP-06     | CON-06         | TC-05     | Critical |
| FR-07   | Any active state → SAFE_MODE                                 | OP-07     | CON-07         | TC-06     | Critical |
| FR-08   | Chamber enters conservation after door/profile checks        | OP-08     | CON-08         | TC-07     | Critical |
| FR-09   | Correction → sensor verification → recovery/protection       | OP-09     | CON-09         | TC-08     | Critical |
| FR-10   | Any logged event                                             | OP-10     | CON-10         | TC-09     | High     |
| HR-02   | Crossing approach → stop or cross safely                     | OP-11     | CON-11         | TC-10     | Critical |
| HR-04   | Active state → LINK_LOST                                     | OP-12     | CON-12         | TC-11     | Critical |
| NFR-03  | 20 concurrent robot sessions                                 | OP-13     | CON-13         | TC-12     | High     |

2.4 Test cases and regression testing

| Test ID | Scenario and steps                                                                   | Expected result                                                                                       | CR requiring rerun                                       |
| ------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| TC-01   | Submit a valid delivery request to an idle robot.                                    | Request is accepted; robot enters NAVIGATING; status is reported and event is logged.                 | —                                                        |
| TC-02   | Submit a command using an unauthenticated account.                                   | Command is rejected, no robot action occurs, and the rejection is logged.                             | CR-03                                                    |
| TC-03   | Detect an obstacle during navigation, then clear it.                                 | Robot enters AVOIDING_OBSTACLE and resumes NAVIGATING only after clearance.                           | —                                                        |
| TC-04   | Reach the destination with safe delivery conditions.                                 | Robot enters DELIVERING only from the permitted navigation state.                                     | —                                                        |
| TC-05   | Inject a critical battery condition during navigation.                               | Delivery is abandoned and the robot returns to the warehouse, subject to the emergency safety policy. | CR-01                                                    |
| TC-06   | Inject critical chamber temperature, battery temperature, or low battery capacity.   | Robot enters SAFE_MODE within 3 seconds of detecting the specified trigger.                           | CR-01                                                    |
| TC-07   | Place a batch in the chamber while the door is open or the profile is missing.       | Conservation does not start until the door is closed and the profile is loaded.                       | —                                                        |
| TC-08   | Trigger an out-of-range temperature, then simulate successful and failed correction. | Sensor readings confirm recovery, or the chamber enters protection mode and alerts the operator.      | —                                                        |
| TC-09   | Issue a command and generate an alarm/state change.                                  | Events are logged with timestamps and relevant IDs.                                                   | CR-03, where command acceptance or rejection is involved |
| TC-10   | Approach a railway crossing with the barrier closed or a train approaching.          | Robot stops and does not enter the crossing.                                                          | —                                                        |
| TC-11   | Disconnect the communication link during navigation.                                 | Robot enters LINK_LOST, follows its safe local policy, rejects remote commands, and logs the failure. | —                                                        |
| TC-12   | Connect 20 robots simultaneously and execute representative status requests.         | All 20 robots connect and the defined service-level targets are met.                                  | CR-02                                                    |

Regression test selection:
- CR-01: rerun TC-05 and TC-06, plus related timing and fault-response tests.
- CR-02: rerun TC-12 and performance tests affected by concurrent connections.
- CR-03: rerun TC-02, TC-09, and authorised-command execution tests.
Any shared code changed by a CR must also trigger the relevant integration and regression tests. A dash means that the test is not directly required by that CR.


2.4 Test cases and regression testing

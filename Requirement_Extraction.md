Task 1: Requirement Extraction

1.1 Functional Requirements (FRs)
| Req. ID | Requirement                                                                                                                                                                   | Type   | Priority | Verification method        |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | -------- | -------------------------- |
| FR-01   | The system shall receive delivery requests containing a valid robot ID, destination, and batch ID.                                                                            | FR     | High     | Functional test            |
| FR-02   | The system shall authenticate each operator and verify role permissions before accepting a robot command.                                                                     | FR     | Critical | Security test              |
| FR-03   | The robot shall navigate to the assigned clinic after accepting a valid delivery request.                                                                                     | FR     | High     | Simulation test            |
| FR-04   | The robot shall suspend navigation and handle an obstacle when an obstacle is detected, then resume navigation after the route is confirmed clear.                            | FR     | Critical | State-transition test      |
| FR-05   | The robot shall deliver a package only after reaching its assigned destination and confirming that delivery conditions are safe.                                              | FR     | Critical | Functional and safety test |
| FR-06   | The robot shall return to the warehouse after delivery or when a critical battery condition requires delivery abandonment.                                                    | FR     | Critical | Scenario test              |
| FR-07   | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds its critical threshold or battery capacity falls below the emergency level.      | Safety | Critical | Timed fault-injection test |
| FR-08   | The chamber shall load the batch identification and required temperature and humidity limits, then start conservation only when the profile is loaded and the door is closed. | FR     | Critical | Integration test           |
| FR-09   | The chamber shall correct out-of-range temperature or humidity and verify recovery through sensor readings before reporting normal conditions.                                | FR     | Critical | Sensor and recovery test   |
| FR-10   | The system shall record robot commands, authentication decisions, chamber alarms, state changes, delivery outcomes, and communication failures with timestamps.               | FR     | High     | Log inspection test        |

1.2 Non-Functional Requirements (NFRs)
 
 Req. ID | Requirement                                                                                                                                                      | Type                        | Priority | Verification method                    |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | -------- | -------------------------------------- |
| NFR-01  | Under normal operating load, the system shall process a received command and return an acknowledgement within 5 seconds for at least 95% of commands.            | NFR — Performance           | High     | Load and latency test                  |
| NFR-02  | The system shall reject every command issued without valid authentication or the required role authorisation.                                                    | NFR — Security              | Critical | Penetration and access-control testing |
| NFR-03  | The system shall support at least 20 simultaneously connected robots serving the pilot clinics.                                                                  | NFR — Scalability           | High     | Concurrent-connection test             |
| NFR-04  | The system shall achieve at least 99.5% service availability during the agreed pilot measurement period, excluding approved maintenance.                         | NFR — Reliability           | High     | Availability monitoring                |
| NFR-05  | The web portal shall allow an authorised operator to locate a robot or batch record within 3 seconds for at least 95% of searches under the 20-robot pilot load. | NFR — Usability/performance | Medium   | Timed usability and load test          |

1.3 High-risk requirements

| Req. ID | High-risk requirement                                                                                                                           | Risk reason                                                    |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| HR-01   | The robot shall enter Safe Mode within 3 seconds of detecting a critical battery or temperature condition.                                      | Delayed response may endanger medicines and the robot.         |
| HR-02   | The robot shall not enter a railway crossing unless the barrier is confirmed open and no train is approaching or present.                       | A violation could cause a collision with a train.              |
| HR-03   | The robot shall not deliver while handling an obstacle, while its chamber door is open, or while chamber conditions are unsafe.                 | Delivery could damage medicines or create an unsafe situation. |
| HR-04   | If communication is lost, the robot shall stop accepting remote commands, enter a defined communication-failure response, and log the incident. | Delayed or missing commands may lead to unsafe actions.        |

|

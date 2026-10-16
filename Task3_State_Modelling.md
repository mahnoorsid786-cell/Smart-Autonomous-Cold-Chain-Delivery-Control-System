Task 3: State Modelling and Transition Verification

3.1 State definitions
A state describes what the robot is doing at a particular time. An event is something that occurs and may cause the robot to change state.

| State ID | State name        | Meaning                                               | Entry condition                                                            | Exit condition                                                    |
| -------- | ----------------- | ----------------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| ST-00    | OFF               | Robot is powered off.                                 | Power is off.                                                              | Power-on event.                                                   |
| ST-01    | IDLE              | Robot is waiting for a valid request.                 | Startup complete or return completed.                                      | Valid delivery request.                                           |
| ST-02    | NAVIGATING        | Robot is travelling to its destination.               | Valid request accepted.                                                    | Obstacle, destination reached, or critical fault.                 |
| ST-03    | AVOIDING_OBSTACLE | Robot is handling an obstacle.                        | Obstacle detected while navigating.                                        | Obstacle cleared and route verified.                              |
| ST-04    | DELIVERING        | Robot is carrying out the delivery procedure.         | Destination reached and delivery safety conditions verified.               | Delivery completed.                                               |
| ST-05    | RETURNING         | Robot is returning to the warehouse.                  | Delivery completed or delivery abandoned for a critical battery condition. | Warehouse reached.                                                |
| ST-06    | SAFE_MODE         | Robot has entered its critical-fault safety response. | Critical battery/temperature trigger or another defined critical fault.    | Fault cleared, sensors verified, and authorised recovery allowed. |
| ST-07    | LINK_LOST         | Communication with Mission Control is unavailable.    | Communication timeout detected.                                            | Link restored and communication integrity verified.               |

3.2 Events and conditions

| Event ID | Event/condition     | Meaning                                                        |
| -------- | ------------------- | -------------------------------------------------------------- |
| EV-01    | PowerOn             | Robot receives power and starts initialisation.                |
| EV-02    | DeliveryRequest     | A valid delivery request is received.                          |
| EV-03    | ObstacleDetected    | An obstacle is detected during navigation.                     |
| EV-04    | ObstacleCleared     | The obstacle is cleared and the route is verified.             |
| EV-05    | DestinationReached  | Robot reaches the assigned destination.                        |
| EV-06    | DeliveryCompleted   | Delivery procedure finishes successfully.                      |
| EV-07    | WarehouseReached    | Robot reaches its warehouse destination.                       |
| EV-08    | CriticalBattery     | Battery capacity falls below the emergency threshold.          |
| EV-09    | CriticalTemperature | Battery or chamber temperature crosses its critical threshold. |
| EV-10    | LinkTimeout         | Communication exceeds the allowed timeout.                     |
| EV-11    | LinkRestored        | Communication is restored and verified.                        |
| EV-12    | FaultCleared        | A critical fault is cleared and recovery checks pass.          |
| EV-13    | CommandReceived     | A remote command arrives.                                      |
| EV-14    | OperatorAuthorised  | Authentication and role checks succeed.                        |
| EV-15    | DoorOpened          | Chamber door opens during conservation.                        |
| EV-16    | SensorFailure       | An essential sensor fails self-check or becomes unreliable.    |

3.3 State transition table

| Current state     | Event/condition                           | Next state                                 | Req. ID      | Valid? / verification evidence                      |
| ----------------- | ----------------------------------------- | ------------------------------------------ | ------------ | --------------------------------------------------- |
| OFF               | PowerOn and startup checks pass           | IDLE                                       | FR-01        | Valid; startup test                                 |
| IDLE              | Valid DeliveryRequest accepted            | NAVIGATING                                 | FR-01, FR-03 | Valid; TC-01                                        |
| IDLE              | DestinationReached without a request      | DELIVERING                                 | HR-03        | Invalid; negative transition test                   |
| NAVIGATING        | ObstacleDetected                          | AVOIDING_OBSTACLE                          | FR-04        | Valid; TC-03                                        |
| AVOIDING_OBSTACLE | ObstacleCleared and route verified        | NAVIGATING                                 | FR-04        | Valid; TC-03                                        |
| NAVIGATING        | DestinationReached and safety checks pass | DELIVERING                                 | FR-05        | Valid; TC-04                                        |
| DELIVERING        | DeliveryCompleted                         | RETURNING                                  | FR-06        | Valid; delivery scenario test                       |
| RETURNING         | WarehouseReached                          | IDLE                                       | FR-06        | Valid; return scenario test                         |
| NAVIGATING        | CriticalBattery                           | RETURNING                                  | FR-06        | Valid if safe to return; TC-05                      |
| NAVIGATING        | CriticalTemperature                       | SAFE_MODE                                  | FR-07        | Valid; TC-06                                        |
| NAVIGATING        | CriticalBattery or CriticalTemperature    | SAFE_MODE                                  | FR-07        | Valid; TC-06; emergency priority applies            |
| AVOIDING_OBSTACLE | CriticalBattery                           | SAFE_MODE                                  | FR-07        | Required; missing in the basic model                |
| DELIVERING        | CriticalBattery or CriticalTemperature    | SAFE_MODE                                  | FR-07        | Required; missing in the basic model                |
| Any active state  | LinkTimeout                               | LINK_LOST                                  | HR-04        | Required; TC-11                                     |
| LINK_LOST         | LinkRestored and integrity verified       | Previous safe state or controlled recovery | HR-04        | Requires recovery test                              |
| SAFE_MODE         | FaultCleared and authorised recovery      | IDLE or safe recovery state                | FR-07        | Requires recovery test                              |
| IDLE              | ObstacleDetected                          | IDLE                                       | FR-04        | Valid if no navigation is active; no motion         |
| OFF               | DeliveryRequest                           | OFF                                        | FR-01        | Invalid; powered-off robot cannot execute a request |

3.4 Extending the model with Safe Mode and Link Lost

- SAFE_MODE: may be entered from any powered state when a critical safety trigger occurs. The robot must inhibit unsafe motion and delivery. Whether it can move toward a safe stopping place must be defined by the safety analysis.
- LINK_LOST: may be entered from any powered state that depends on Mission Control communication. It must follow a locally defined safe policy; it must not wait indefinitely for a remote command.
- Recovery: neither state should be exited merely because a signal or command arrives. Recovery requires verified fault clearance, valid sensor information, and any required operator authorisation.

- 3.5 Behavioural verification and defects
  
(a) Can IDLE → DELIVERING happen?
No. The transition is invalid because the robot must first accept a valid request, navigate to the destination, and verify delivery conditions. It violates FR-03 and FR-05.

(b) What if AVOIDING_OBSTACLE has no transition back?
The robot may remain stuck indefinitely. This is a liveness defect: it cannot resume navigation even after the obstacle clears. The transition required by FR-04 must be present and verified by TC-03. A timeout or unresolved obstacle must lead to a safe stop or escalation, not forced delivery.

(c) Can AVOIDING_OBSTACLE → DELIVERING occur?
Not directly. It must return to NAVIGATING, then establish that the destination has been reached and delivery conditions are safe. Otherwise, the robot could deliver while still handling an obstacle, violating HR-03.

(d) What happens on CriticalBattery during AVOIDING_OBSTACLE or DELIVERING?
The original basic model omits these transitions. That is a safety defect because a critical battery condition could be ignored in either state.
The corrected model must give the critical trigger higher priority and enter SAFE_MODE within the required 3 seconds. The subsequent safe recovery policy must determine whether a controlled return is possible. The robot must not blindly continue delivering or initiate movement that cannot be performed safely.
Other defects to test include stale commands after a link interruption, destination events arriving out of order, repeated delivery requests, and recovery without fresh sensor verification.



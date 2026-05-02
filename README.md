Furuta Pendulum — Design, Implementation and Control
A fully functional rotating inverted pendulum (Furuta Pendulum) built from scratch. The system combines Lagrangian mechanics, LQR optimal control, real-time embedded firmware, and a custom Qt GUI — all running on a Raspberry Pi Pico at 1 kHz.

Graduation project — Ankara Yıldırım Beyazıt University, Mechanical Engineering, May 2025
Supervised by Assoc. Prof. Tolga Özaslan


What this is
The Furuta Pendulum is a classic benchmark in nonlinear control: a pendulum mounted on the end of a horizontally rotating arm, where the only control input is the torque applied to the base. The pendulum angle cannot be directly actuated — the controller must indirectly stabilize it through base rotation.
This makes it directly analogous to real-world problems: rocket attitude control, bipedal locomotion, and self-balancing platforms.

Results
MetricValueSettling time~1.8 sOvershoot< 4°Steady-state error< 0.5°Stable recovery range±30°Control loop frequency1 kHzDisturbance rejection~1–2 s
The system maintained balance continuously for >5 minutes with no oscillation buildup under repeated trials (±5% variation in settling time across 10 runs).

System overview
[Encoder] ──→ [Raspberry Pi Pico] ──→ [TB6600 Driver] ──→ [NEMA 23 Motor]
                      │                                            │
                  LQR Control                              Rotary Base Arm
                  u = −Kx                                        │
                      │                                     Pendulum Rod
                  [Qt GUI] ←── USB Serial ───────────────────────┘
Hardware:

Microcontroller: Raspberry Pi Pico (ARM Cortex-M0+, 133 MHz)
Actuator: NEMA 23 stepper motor + TB6600 driver (1/32 microstepping)
Sensor: E50S8-500-3-T-24 incremental encoder (500 PPR, quadrature)
Structure: Aluminum 6061-T6 frame, PLA/SLA 3D-printed mounts


Mathematical modeling
Full system dynamics were derived using Lagrangian mechanics.
Generalized coordinates:

θ(t) — base arm angle (controlled via motor torque τ)
α(t) — pendulum angle from vertical (unactuated)

Kinetic energy:
T = ½·Jr·θ̇² + ½·m·(ẋ² + ẏ² + ż²) + ½·Jp·α̇²
where the pendulum CoM velocity components are derived via the chain rule in 3D Cartesian coordinates.
Potential energy:
V = m·g·L·cos(α)
Applying the Euler–Lagrange equations yields two coupled nonlinear ODEs. The system is then linearized around the upright equilibrium (α=0, α̇=0) via first-order Taylor expansion, producing the state-space form:
ẋ = Ax + Bu,   x = [θ, α, θ̇, α̇]ᵀ,   u = τ

Control design — LQR
LQR was selected for its optimality guarantees on the linearized model and minimal computational overhead on embedded hardware.
Cost function minimized:
J = ∫₀^∞ (xᵀQx + uᵀRu) dt
Gain matrix computed offline (MATLAB lqr()) and hardcoded on the Pico:
u(t) = −K·x(t)
Q/R tuning rationale:

High weight on α → fast pendulum recovery
Moderate weight on θ → smooth base motion
Low weight on velocities → no aggressive damping
Small R → allow sufficient control authority

The Algebraic Riccati Equation (CARE) is solved once offline; only a matrix-vector multiply runs on the microcontroller.

Embedded implementation

Control loop: Hardware timer interrupt at 1 kHz (1 ms period)
Encoder reading: GPIO interrupt-driven quadrature decoding
Velocity estimation: Discrete differentiation + first-order low-pass filter
Motor output: STEP/DIR pulse generation to TB6600
Communication: UART over USB serial to Qt GUI at 20–50 Hz

Key engineering problems solved:

Encoder noise at high angular speeds → digital low-pass filtering
Mechanical misalignment causing encoder drift → custom 3D-printed shaft mounts with tight concentricity tolerances
Torque saturation → software-level clipping before motor command


Qt GUI
A custom desktop application handles real-time monitoring and control.
Features:

Live plots of α, θ, θ̇, α̇
Start / Stop / Reset / Emergency Stop buttons
COM port auto-detection and connection management
Serial packet validation and error handling
Status indicators (controller active, pendulum balanced, safety limit)

Architecture: Event-driven with QTimer-based polling, QSerialPort for UART, QCustomPlot for real-time visualization.

Repository structure
furuta-pendulum/
├── embedded/          ← Raspberry Pi Pico C++ firmware (control loop, encoder, UART)
├── matlab/            ← Lagrangian derivation, linearization, LQR gain computation
├── qt-gui/            ← Qt desktop application source
└── docs/              ← CAD renders, circuit diagrams, experimental data
RepoContentsInverted-PendulumEmbedded C++ firmwareInverted-pendulum-QT-GUIQt GUI sourceInverted-Pndulum-matlab-codesMATLAB modeling & LQR

Bill of materials
ComponentModelCost (₺)MicrocontrollerRaspberry Pi Pico400Stepper motorNEMA 23, 1.8°/step2000Motor driverTB6600800EncoderE50S8-500-3-T-24, 500 PPR3600Power supply12V 5A DC500Frame & mountsAluminum + 3D printed3000Electronics miscWiring, PCB, caps400Total~10,300 ₺

Future work

Swing-up control — energy-based controller to bring pendulum from rest to upright before LQR takes over
Kalman filter — replace discrete differentiation with observer-based state estimation
Nonlinear control — sliding mode or feedback linearization for recovery beyond ±35°
Higher-resolution sensing — 14-bit absolute encoder or IMU fusion
Torque-controlled actuator — closed-loop servo for direct τ mapping


Full report
The complete 90-page graduation report (mathematical derivations, circuit schematics, experimental data) is available in the project documentation folder.

Built with C++, MATLAB/Simulink, Qt — running on Raspberry Pi Pico

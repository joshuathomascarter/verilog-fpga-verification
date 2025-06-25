FSM Entropy Overlay Module
Project Overview
The FSM Entropy Overlay Module is a critical hardware component designed to bolster the resilience and security of advanced CPU architectures, specifically within the ARCHON hybrid CPU system. This module implements a sophisticated Finite State Machine (FSM) that functions as an adaptive control layer, dynamically mitigating potential threats and ensuring operational integrity in the face of unpredictable or anomalous conditions.

By continuously monitoring system health indicators such as internal entropy, architectural hazards, and leveraging real-time machine learning predictions, this module can intelligently trigger defensive responses like system stalls, pipeline flushes, or critical security locks.

This module has been fully synthesized and rigorously simulated using Quartus Prime and ModelSim, confirming its readiness for FPGA pin assignment and seamless integration with a live instruction core.

Features
Dynamic System Adaptation: Adjusts operational state in real-time based on system health.

Multi-layered Threat Detection: Integrates Machine Learning predictions, entropy analysis, and hazard flags.

High-Priority Overrides: Responds instantly to critical analog and quantum override signals.

Context-Aware Responses: Adapts behavior based on the type of instruction being executed.

Detailed Logging: Provides output logs for entropy scores and instruction types during state transitions, crucial for post-incident analysis.

Operational States
The FSM operates in one of four distinct states, each designed to address different levels of system risk:

STATE_OK (2'b00): Normal operation. The system is functioning as expected without any detected anomalies.

STATE_STALL (2'b01): Halts execution. Initiated to prevent potential issues, allowing time for diagnosis, resolution, or re-evaluation.

STATE_FLUSH (2'b10): Clears pipelines/buffers. Typically invoked in response to detected data corruption or irrecoverable internal states, sanitizing the system.

STATE_LOCK (2'b11): Secure, unchangeable state. The most critical state, indicating a severe security breach or integrity compromise. Requires external reset to recover.

Inputs
The FSM's decision-making is driven by the following input signals:

clk (wire): Clock signal.

rst_n (wire): Asynchronous active-low reset.

ml_predicted_action (2-bit wire): Machine Learning model's suggested action (e.g., ML_OK, ML_STALL, ML_FLUSH, ML_LOCK).

internal_entropy_score (8-bit wire): A measure of randomness/unpredictability within the system.

internal_hazard_flag (wire): Indicates an architectural hazard.

analog_lock_override (wire): External, high-priority signal to force STATE_LOCK.

analog_flush_override (wire): External, high-priority signal to force STATE_FLUSH.

classified_entropy_level (2-bit wire): Pre-classified severity of entropy (ENTROPY_LOW, ENTROPY_MID, ENTROPY_CRITICAL).

quantum_override_signal (wire): Highest priority override, forcing STATE_LOCK.

instr_type (3-bit wire): Categorization of the currently executing instruction (e.g., INSTR_TYPE_ALU, INSTR_TYPE_LOAD, INSTR_TYPE_BRANCH).

Outputs
The module provides these outputs for external monitoring and logging:

fsm_state (2-bit reg): The current operational state of the FSM.

entropy_log_out (8-bit reg): Logs the internal_entropy_score when a state transition occurs.

instr_type_log_out (3-bit reg): Logs the instr_type when a state transition occurs.

These output logs serve as a crucial transcript of the FSM's decisions and the contextual data that led to those decisions, providing invaluable insights for debugging and system analysis.

How It Works
The FSM operates on a synchronous clock and an asynchronous reset. Its next state is determined by a combinational logic block that prioritizes inputs hierarchically:

Quantum Override: Highest priority, immediately forces STATE_LOCK.

Analog Overrides: analog_lock_override then analog_flush_override take precedence.

Classified Entropy Level: If no high-priority overrides are active, the system assesses the classified_entropy_level:

ENTROPY_CRITICAL: Triggers aggressive responses (e.g., STALL for branches/jumps, FLUSH for memory operations) based on instr_type.

ENTROPY_MID: Leads to more conservative STALL actions, often dependent on the current state or internal_hazard_flag.

ENTROPY_LOW (or uncategorized): Primarily defers to ml_predicted_action. If ML suggests ML_OK, the module then checks for internal_entropy_score > ENTROPY_HIGH_THRESHOLD or internal_hazard_flag to initiate a STALL.

State-Specific Transitions: Transitions from STATE_STALL or STATE_FLUSH back to STATE_OK (or STATE_STALL from STATE_FLUSH) occur if ml_predicted_action is ML_OK, internal_hazard_flag is clear, and internal_entropy_score is low. The STATE_LOCK is terminal, requiring an external reset.

Deployment
This FSM Entropy Overlay Module is a core component within the ARCHON hybrid CPU system. It has been:

Synthesized: Using Intel Quartus Prime, verifying its hardware implementation.

Simulated: Extensively tested via ModelSim, demonstrating correct logical behavior under various scenarios (refer to fsm_entropy_overlay_tb.sv for test bench details).

The module is now prepared for FPGA pin assignment and direct integration into the ARCHON's live instruction core, forming the critical "hazard reflex" control backbone.

Future Enhancements (Ideas for ARCHON Development)
Dynamic Threshold Adjustment: Implement adaptive thresholds for entropy based on historical data or system load.

External Configuration: Allow external configuration of FSM parameters (e.g., entropy thresholds) for greater flexibility.

State History Buffer: Introduce a deeper logging buffer for more extensive post-mortem analysis.

ML Model Feedback Loop: Explore ways for the FSM's logged data to provide feedback to the ML model for continuous improvement.

License
This project is open-sourced under the MIT License. See the LICENSE file for details.

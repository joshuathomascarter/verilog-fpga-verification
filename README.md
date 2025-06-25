# FSM Entropy Overlay Module

## Project Overview

The **FSM Entropy Overlay Module** is a critical hardware component designed to bolster the resilience and security of advanced CPU architectures, specifically within the **ARCHON hybrid CPU system**.

This module implements a sophisticated Finite State Machine (FSM) that functions as an adaptive control layer, dynamically mitigating potential threats and ensuring operational integrity in the face of unpredictable or anomalous conditions.

By continuously monitoring system health indicators—such as internal entropy, architectural hazards, and leveraging real-time machine learning predictions—the module can intelligently trigger defensive responses like system stalls, pipeline flushes, or critical security locks.

This module has been fully synthesized and rigorously simulated using **Quartus Prime** and **ModelSim**, confirming its readiness for FPGA pin assignment and integration with a live instruction core.

---

## Features

- **Dynamic System Adaptation**: Adjusts operational state in real-time based on system health.
- **Multi-layered Threat Detection**: Integrates machine learning predictions, entropy analysis, and hazard flags.
- **High-Priority Overrides**: Responds instantly to critical analog and quantum override signals.
- **Context-Aware Responses**: Adapts behavior based on the type of instruction being executed.
- **Detailed Logging**: Provides logs for entropy scores and instruction types during state transitions.

---

## Operational States

The FSM operates in one of four distinct states:

- `STATE_OK (2'b00)`:  
  Normal operation. System is functioning normally.

- `STATE_STALL (2'b01)`:  
  Halts execution to prevent potential issues. Allows time for diagnosis or reevaluation.

- `STATE_FLUSH (2'b10)`:  
  Clears pipelines and buffers. Used in cases of data corruption or unrecoverable internal states.

- `STATE_LOCK (2'b11)`:  
  Secure, unchangeable state. Indicates a severe breach. Requires external reset.

---

## Inputs

The FSM's decisions are based on the following input signals:

- clk                      // Clock signal
- rst_n                   // Asynchronous active-low reset
- ml_predicted_action     // 2-bit: Machine learning suggested action
- internal_entropy_score  // 8-bit: Measures system unpredictability
- internal_hazard_flag    // Architectural hazard flag
- analog_lock_override    // External high-priority signal -> STATE_LOCK
- analog_flush_override   // External high-priority signal -> STATE_FLUSH
- classified_entropy_level// 2-bit: ENTROPY_LOW, MID, CRITICAL
- quantum_override_signal // Highest priority -> STATE_LOCK
- instr_type              // 3-bit: Instruction type (ALU, LOAD, BRANCH)

---

These signals are available for external monitoring and logging:
## Outputs

These signals are available for external monitoring and logging:
---

## Outputs

fsm_state           // 2-bit: Current FSM operational state
entropy_log_out     // 8-bit: Logs entropy score during state transitions
instr_type_log_out  // 3-bit: Logs instruction type during transitions

## How It Works
The FSM is clock-driven with an asynchronous reset. Next-state logic prioritizes input signals hierarchically:

Quantum Override
- Immediate transition to STATE_LOCK.

Analog Overrides
- analog_lock_override takes precedence.
- If not active, analog_flush_override is checked.

Classified Entropy Level
If no high-priority overrides are active, the FSM evaluates classified_entropy_level:

ENTROPY_CRITICAL:
- Triggers STALL or FLUSH based on instr_type.

ENTROPY_MID:
- Leads to conservative STALL actions, especially if internal_hazard_flag is active.

ENTROPY_LOW:
- Defers to ml_predicted_action.
- If ML suggests ML_OK but entropy is still high or a hazard flag is raised, the system will still STALL.

## State-Specific Transitions
FSM returns from STATE_STALL or STATE_FLUSH to STATE_OK if:
- ml_predicted_action is ML_OK
- internal_entropy_score is low
- internal_hazard_flag is clear
Note: STATE_LOCK is terminal and requires an external reset to recover.

## Deployment
This module is a core control component in the ARCHON hybrid CPU system.
- Synthesized with: Intel Quartus Prime
- Simulated via: ModelSim
- Verified against: Full instruction core
- Testbench file: fsm_entropy_overlay_tb.sv

----

## Future Enhancements (ARCHON Roadmap Ideas)
- Dynamic Threshold Adjustment: Adjust entropy thresholds based on historical runtime behavior.
- External Configuration Support: Enable runtime configurability of FSM logic.
- State History Buffer: Add deeper logging for extended post-mortem analysis.
- ML Feedback Loop: Feed FSM log data back into the ML model for continuous retraining and adaptation.

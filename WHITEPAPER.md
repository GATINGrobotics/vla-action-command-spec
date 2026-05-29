# The AI-to-Actuator Trust Boundary for Vision-Language-Action Robots

## Executive Summary

Vision-Language-Action systems are moving robots from scripted automation toward general physical autonomy. A VLA model can perceive a scene, interpret a natural-language objective, reason about available objects, and propose an action for a robot body. That creates a powerful new capability, but it also creates a new safety and governance problem: the model output is no longer merely text, code, or a recommendation. It may become physical motion.

EdgeRobotics proposes an AI-to-actuator trust boundary for VLA-controlled robots. Under this architecture, a VLA model may propose an action, but the action is not executable until a downstream gate validates the command against robot state, environment context, role, policy, capability limits, and actuator constraints.

The core principle is simple:

> A robot may propose an action, but it should not execute until the action is validated.

## 1. The Problem

Modern AI safety work often focuses on model prompts, hallucinations, content filtering, or software-agent permissions. Physical AI requires an additional layer. A robot action command can move a limb, apply force, grip an object, open a door, board a vehicle, enter a building, or interact with a person.

For a VLA robot, the relevant failure mode is not only that the model “says” something wrong. The relevant failure mode is that the robot executes a physically unsafe, unauthorized, contextually inappropriate, or legally unaccountable action.

Examples include:

* A robot arm moves while a human is too close.
* A mobile robot continues a mission with insufficient battery reserve.
* A humanoid attempts a task outside its authorized role.
* A robot hand applies excessive grip force to a fragile or human-adjacent object.
* A robot receives a plausible but unsafe action proposal from a compromised or hallucinating model.
* A fleet robot enters a location without infrastructure authorization.
* A robot actuator continues executing after a policy, ownership, or emergency-stop condition changes.

In each case, a model-level answer is not enough. The system needs a downstream validation boundary between AI intention and physical execution.

## 2. The AI-to-Actuator Trust Boundary

The AI-to-actuator trust boundary is a control layer positioned between a VLA model and a robot execution stack.

A simplified architecture is:

```text
VLA Model
  ↓
Proposed Action Command
  ↓
Action Gate / Policy Validator
  ↓
Gate Decision: ALLOW, DENY, or MODIFY
  ↓
Approved Command
  ↓
Robot Controller / Actuator Layer
```

The VLA model generates a proposed action. The proposed action is serialized as an action command. A gate evaluates the command against current robot context, environment context, policy constraints, and capability limits. The gate then returns one of three decisions:

* `ALLOW`: the command may proceed.
* `DENY`: the command is blocked.
* `MODIFY`: the command may proceed only after limits or parameters are reduced or changed.

This structure converts robot action from an unconstrained model output into a validated, policy-bound execution request.

## 3. Reference Command Interface

A VLA action command should identify the action, robot, source model, target, and execution limits. A minimal command may include:

```json
{
  "action_id": "cmd-001",
  "robot_id": "dor-demo-01",
  "timestamp": "2026-05-28T12:00:00Z",
  "source_model": "example-vla",
  "action_type": "move_arm",
  "target": {
    "frame": "base_link",
    "position": [0.25, 0.10, 0.45],
    "orientation": [0, 0, 0, 1]
  },
  "limits": {
    "max_velocity_mps": 0.3,
    "max_force_n": 20,
    "max_torque_nm": 5
  }
}
```

A corresponding robot context may include:

```json
{
  "robot_state": {
    "battery_percent": 72,
    "emergency_stop": false,
    "current_role": "warehouse_assistant"
  },
  "environment": {
    "human_distance_m": 1.8
  },
  "policy": {
    "allowed_actions": ["move_arm", "grasp", "release"],
    "denied_actions": ["strike", "throw", "self_modify"],
    "min_human_distance_m": 0.6,
    "min_battery_percent": 15,
    "max_velocity_mps": 0.5,
    "max_force_n": 25,
    "max_torque_nm": 8
  }
}
```

The gate decision may be:

```json
{
  "decision": "ALLOW",
  "reason": "command satisfies reference policy",
  "action_id": "cmd-001",
  "audit_id": "gate-20260528-0001"
}
```

Or, when a policy is violated:

```json
{
  "decision": "DENY",
  "reason": "human proximity below required threshold",
  "action_id": "cmd-001",
  "audit_id": "gate-20260528-0002"
}
```

## 4. Why the Gate Should Be Downstream of the VLA Model

The validation gate should not be treated as merely another prompt, system instruction, or model-side refusal rule. For physical AI, the gate should be downstream from the model because the gate is closer to the robot’s actual state and execution layer.

The VLA model may be powerful, but it may not have authoritative access to:

* real-time emergency-stop state;
* actuator-level limits;
* force and torque envelopes;
* human-proximity sensors;
* ownership or custody status;
* infrastructure access permissions;
* role authorization;
* battery reserve;
* network availability;
* maintenance status;
* local legal or premises-specific constraints.

A separate action gate can evaluate the proposed action using real-time context and deterministic policy logic. The result is not merely “the model thinks this is safe.” The result is “the command has been validated against the current execution context.”

## 5. Policy Dimensions

A practical VLA action gate may evaluate several dimensions.

### Safety

The gate can block commands when a human is too close, when an emergency stop is active, when a force limit is exceeded, or when a proposed action is categorically unsafe.

### Capability

The gate can restrict actions to the robot’s actual physical capabilities, such as arm reach, hand load, torque limits, locomotion mode, or available sensors.

### Role

The gate can verify that the robot is acting in an authorized role, such as warehouse assistant, courier, maintenance agent, caregiver, or security agent.

### Context

The gate can evaluate whether the environment permits the action, such as whether the robot is in a permitted workspace, building, vehicle, elevator, charging area, or human-shared zone.

### Resource State

The gate can deny or modify missions when battery, compute, network, maintenance, or service-account conditions are insufficient.

### Auditability

The gate can create a record showing why the command was allowed, denied, or modified. This is important for debugging, compliance, insurance, fleet management, and liability analysis.

## 6. Open Interface, Commercial Enforcement

The open-source layer should define an interface and a reference gate. That layer can include:

* VLA action-command schema;
* robot-context schema;
* gate-decision schema;
* basic policy validator;
* example command and context payloads;
* audit output format;
* simple developer demos.

The production enforcement layer is different. A commercial actuator-firewall system may include:

* cryptographic actuator release tokens;
* time-bound actuator permissions;
* motor-driver enforcement;
* RTOS or firmware hooks;
* hardware-rooted identity;
* emergency revocation;
* ownership and custody gating;
* infrastructure access gating;
* resource and service-account gating;
* identity and role gating;
* multi-gate actuation arbitration;
* fleet-level policy management.

This separation allows the interface to be open while preserving commercial and patent-reserved enforcement layers.

## 7. Relationship to EdgeRobotics

EdgeRobotics focuses on the trust boundary for physical AI. The company’s position is that VLA-controlled robots need a validation layer between proposed action and physical execution.

The public repositories provide a narrow open interface and reference implementation. They are intended for research, prototyping, interoperability, and developer education. They are not production actuator-enforcement systems and are not safety-certified.

Commercial implementations may require deeper enforcement, including actuator-level validation, cryptographic authorization, revocation, audit logging, and multi-gate arbitration.

## 8. Conclusion

VLA models make robots more general, but generality increases the need for downstream validation. The physical world requires more than model confidence. It requires enforceable boundaries.

The AI-to-actuator trust boundary provides that boundary.

A VLA robot should not execute merely because a model proposed an action. It should execute only after the proposed action has been validated against policy, context, role, state, and actuator constraints.

That is the core purpose of VLA Action Gate and the broader EdgeRobotics trust-boundary architecture.


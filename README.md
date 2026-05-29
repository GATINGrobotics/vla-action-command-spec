# VLA Action Command Spec

An open interface and reference gate for validating VLA robot action commands before physical execution.

## Overview

VLA Action Command Spec defines a proposed interface between a Vision-Language-Action model and a downstream robot execution layer.

A VLA model may propose an action, but the action should not be executable until a gate validates the command against robot state, environment context, role, policy, and capability constraints.

## What This Repository Provides

This repository provides:

- A proposed JSON schema for VLA-generated robot action commands.
- A proposed JSON schema for robot context and policy state.
- A proposed JSON schema for gate decisions.
- Example command, context, and decision payloads.

## What This Repository Does Not Provide

This repository is not a production actuator-enforcement system.

It does not include motor-driver enforcement, cryptographic actuator release tokens, ownership or custody gating, infrastructure gating, resource gating, identity or role gating, safety certification, or multi-gate actuation arbitration.

Commercial implementations may require a separate EdgeRobotics license.

## Core Concept

A VLA model proposes an action. The action command is serialized using the command schema. A downstream validation gate evaluates the command against context and policy. The gate returns one of three decisions:

- `ALLOW`
- `DENY`
- `MODIFY`

Only allowed or modified commands should be forwarded to a robot execution layer.

## Status

This repository is an early public interface specification. It is intended for research, discussion, prototyping, and interoperability testing.

# Solaria Hub Vision

> **Status:** Living Document
> **Date:** May 25, 2026

## The Core Idea: An Agentic AI Overlay

Solaria Hub is not designed to replace existing coding platforms. Instead, it is strategically positioned as an **infrastructure layer**—an intelligent overlay that sits on top of established environments like MIT App Inventor, Scratch™, MakeCode®, and Arduino IDE.

The goal is to augment these platforms with an agentic AI layer that deeply understands the **hardware × software matrix**. By operating as an overlay, Solaria ensures that users can remain in the coding environments they already know and love, while gaining powerful new capabilities to interact with physical hardware.

## Key Capabilities

The Solaria Hub overlay introduces three primary capabilities to existing platforms:

### 1. Vibe Coding
Solaria enables natural language translation directly into blocks or code. Students and makers can express their intent—such as "Drive forward until you see a red object"—and the AI layer will generate the appropriate logic in the host platform's native format.

### 2. GUI Electronics Configuration
Configuring hardware is often the most frustrating part of robotics. Solaria provides a visual, drag-and-drop interface for setting up microcontrollers, sensors, and motors. It handles the complex pin mappings and driver configurations behind the scenes, presenting a clean UI to the user.

### 3. Agentic AI Assistance
At its core, Solaria is an intelligence layer capable of reasoning about physical components. It understands what hardware is connected, what capabilities that hardware possesses, and how to generate the correct Solaria Standard Protocol (SSP) commands to control it. This agentic assistance works across any combination of supported hardware and software.

## Why This Approach? (Augment, Don't Compete)

The educational technology landscape is highly fragmented. Creating yet another standalone IDE would only add to this fragmentation and force educators to learn a new tool.

By building Solaria as an infrastructure layer, we adopt a philosophy of **augmentation over competition**. 
- **For users:** Zero switching costs. They use the tools they already know.
- **For platforms:** Solaria acts as a powerful feature upgrade, making every existing platform a potential integration partner.
- **For hardware:** Solaria bridges the gap between cheap, ubiquitous hardware and high-level visual programming.

## Supported Targets

### Software Platforms
Solaria is designed to integrate with the most widely used educational and maker platforms:
- **MIT App Inventor**
- **Scratch™**
- **MakeCode®**
- **Arduino IDE**
- **Python**
- **Web (JavaScript)**
- **MicroBlocks**

### Hardware Platforms
Solaria supports a wide range of hardware through a hybrid architecture defined in the [ARCHITECTURE.md](ARCHITECTURE.md):
- **Type 1 (Native SSP):** Open hardware that can be flashed with Solaria firmware (e.g., ESP32, BBC micro:bit™, StackChan, Raspberry Pi, CyberBrick).
- **Type 2 (Protocol Bridges):** Closed hardware that requires a translation library (e.g., LEGO® SPIKE™ Prime, LEGO Powered Up, Sony® toio™, UBTECH® uGot).

## Proof of Concept: Bridge Suite

The first implementation of this vision is the **Bridge Suite**, which successfully connects MIT App Inventor to the LEGO® SPIKE™ Prime hub via Bluetooth. 

This Gen 1 MVP proves the core architecture: a user can open App Inventor, connect to a closed-ecosystem robot using a Solaria wrapper, and control it using standard blocks. It serves as the foundation upon which the AI overlay will be built.

## Sustainability: Open Core, Hosted Cloud

Solaria follows an **open-core** model. The platform, hardware configurations, integrations, and protocol libraries are fully open-source. Anyone can run Solaria locally, contribute extensions, and build on the ecosystem without restriction.

The AI-powered features (vibe coding, agentic assistance) require LLM inference, which has real token costs. To address this sustainably:

- **Community Edition (Free):** Bring Your Own Key (BYOK). Users plug in their own LLM API key (OpenAI, Anthropic, local models). Full functionality, zero cost to Solaria.
- **Hosted Edition (Paid, for schools):** Solaria-managed API with curated prompts, school dashboard, usage analytics, fleet management, and curriculum integration. One invoice, no API key friction for teachers.

This is the GitLab/Supabase model applied to educational robotics: open core drives community adoption and contributions; the hosted tier serves institutions that need simplicity and support.

The tools remain accessible to those who need them most, while providing the resources necessary to expand the ecosystem.

---
title: Simulation and enterprise end-to-end
layout: home
parent: Enterprise features
nav_order: 7
---


# Simulation and Enterprise End-to-End Testing

> If interested, contact sales for more information on pricing.

The **Simulation and Enterprise End-to-End Testing** feature in the Signaturgruppen Broker provides a robust way to integrate with the broker's OIDC interface while simulating external dependencies. This feature enables seamless testing and operation in scenarios where external integrations to Signaturgruppen Broker may not be online or fully functional.

---

## Overview

By adding the top-level `simulation` parameter to the broker's OIDC interface, the Signaturgruppen Broker simulates all external integrations, including API calls to **MitID** and **NemLog-In** backends. This ensures reliable and efficient testing, allowing developers and enterprises to validate their integrations even in offline scenarios.

### Key Simulation Modes

1. **UI Mode** (`simulation=ui`):  
   In this mode, external UIs such as the MitID Client are simulated, providing a complete user interface experience for flows.

2. **No UI Mode** (`simulation=no-ui`):  
   This mode skips UI elements entirely, streamlining the process for backend testing or automated workflows.

---

## Features and Benefits

### 1. **Simulated External Integrations**
   - External systems like MitID and NemLog-In are fully simulated using real data.
   - Eliminates dependency on live services, ensuring uptime and enabling offline testing.

### 2. **Improved Performance**
   - Simulation flows are faster than live flows:
     - The MitID Client’s obfuscation and wait screens are skipped.
     - No user credentials are required in simulated setups.
   - Faster testing cycles and efficient debugging.

### 3. **Reliable End-to-End Testing**
   - Stabilized flows and UI ensure that end-to-end tests run reliably and succeed consistently.
   - Enables predictable and repeatable testing scenarios.

### 4. **Seamless Integration Toggling**
   - The output with or without the simulation parameters is identical, allowing easy toggling between live and simulated setups.
   - Ensures reliable testing without additional integration adjustments.

---

## Simulation Parameters

Simulation parameters are set as space-delimited values in the query URL. These parameters work for both `ui` and `no-ui` modes, providing flexibility in configuring test scenarios. 

### Available Parameters

1. **MitID UUID (`uuid`)**  
   Use a unique identifier for the MitID user being simulated.  
   Example:  
   `uuid:GUID`

2. **CPR Number (`cpr`)**  
   Simulate a specific CPR number for the user.  
   Example:  
   `cpr:1234567890`

3. **MitID Erhverv UUID (`mitid_erhverv_uuid`)**  
   Use a unique identifier for a MitID Erhverv user.  
   Example:  
   `mitid_erhverv_uuid:GUID`

### Parameter Usage Examples

#### UI Mode Example
```plaintext
https://pp.netseidbroker.dk/op/connect/authorize?client_id=CLIENT_ID&response_type=...&simulation=ui uuid:GUID
```

#### No UI Mode Example
```plaintext
https://pp.netseidbroker.dk/op/connect/authorize?client_id=CLIENT_ID&response_type=...&simulation=no-ui uuid:GUID mitid_erhverv_uuid:GUID
```

By combining these parameters, you can tailor your simulation scenarios to match specific use cases or workflows.

---

## How It Works

### Enabling Simulation
Add the `simulation` parameter and any additional options to the OIDC interface request URL.

### Use Cases
- **Development and Testing:** Test integrations without reliance on external services.
- **Performance Testing:** Validate and optimize workflows without the delays inherent in live systems.
- **Continuous Integration/Deployment:** Run stable, fast, and reliable end-to-end tests in automated pipelines.

---

## Compatibility and Consistency

- **Identical Outputs:**  
  Any integration using simulation can expect the same output as when using live external integrations. This ensures:
  - Consistent behavior between testing and production.
  - Simplified debugging and validation.

- **Fail-Safe Testing:**  
  Simulated tests are designed to work seamlessly with the same integration code used for live scenarios. This ensures a high degree of confidence when toggling between simulation and live modes.

---

## Advantages for Enterprises

The Simulation and Enterprise End-to-End Testing feature provides a powerful toolkit for enterprises by enabling:

1. **Reliable Uptime:** Operate even when external dependencies are unavailable.
2. **Cost-Effective Testing:** Reduce reliance on costly or slow external services during development.
3. **Scalable Testing Environments:** Run multiple test scenarios in parallel without impacting live systems.


The Signaturgruppen Broker's Simulation and End-to-End Testing feature is a core enabler for efficient, reliable, and scalable integrations. By leveraging the flexible simulation parameters, developers and enterprises can achieve testing efficiency and predictability.

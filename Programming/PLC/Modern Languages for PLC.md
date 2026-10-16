If you mean **“modern” beyond classic IEC 61131-3 ladder/FBD**, here’s what’s current in industry:

## Modern (or modernized) options

1. **Structured Text (ST), IEC 61131-3 v3**
    
    - Today’s ST is far more “modern” than old PLC code: **interfaces, methods, OOP-style function blocks, namespaces, unit testing** (vendor-specific).
        
    - Best all-around choice for new PLC logic when you want readable, testable code.
        
2. **IEC 61499 (function blocks for distributed control)**
    
    - Componentized, event-driven FBs across devices; good for distributed/IIoT systems.
        
    - Stacks: Eclipse 4diac (FORTE), nxtControl, etc.
        
3. **C / C++ in realtime modules (vendor-specific)**
    
    - **Beckhoff TwinCAT 3**: write **hard-real-time** tasks in **C++** alongside ST.
        
    - **Phoenix Contact PLCnext**: supports **C/C++**, **.NET**, **Python** modules on a Linux-based controller.
        
    - Great when you need heavy math, custom protocols, or leverage existing C libs.
        
4. **C# / .NET (non-hard-RT or vendor RT layers)**
    
    - HMI/SCADA, services, or soft-RT pieces (e.g., TwinCAT HMI, PLCnext, Ignition modules).
        
5. **Python / Lua (scripting on edge gateways / Linux PLCs)**
    
    - Fast prototyping, glue code, ML inferencing, data shaping—**not** for critical fast control loops.
        
6. **Rust / Go (edge services & gateways)**
    
    - Build robust microservices for telemetry, OPC UA/MQTT bridges, protocol translation. Usually paired with a PLC, not replacing it for sub-ms control.
        
7. **Model-based codegen**
    
    - **MATLAB/Simulink/Stateflow** → generate ST or C for advanced control/PID/state machines; nice for aerospace/automotive-style workflows.
        
8. **gRPC/OPC UA/MQTT + Node-RED**
    
    - Not PLC languages per se, but **modern integration stacks** widely used for IIoT, dashboards, and orchestration.
        

---

## When to use what (quick guide)

|Need|Good choice|
|---|---|
|Most new PLC apps, readable & testable|**Structured Text (modern IEC v3)**|
|Distributed, componentized control across devices|**IEC 61499**|
|Hard-real-time + reuse C libs|**C/C++ module (TwinCAT 3, PLCnext)**|
|HMI, services, business logic|**C#/.NET**|
|Edge glue, data science, quick protos|**Python**|
|Robust edge microservices, protocol bridges|**Rust / Go**|
|Math-heavy control with design tools|**Simulink/Stateflow → codegen**|

---

## Tiny taste of “modern” ST (OOP-style)

```pascal
INTERFACE IMotor
    METHOD Start;
    METHOD Stop;
    PROPERTY Running : BOOL READ;
END_INTERFACE

FUNCTION_BLOCK Motor IMPLEMENTS IMotor
VAR
    _running : BOOL;
END_VAR

METHOD Start
    _running := TRUE;

METHOD Stop
    _running := FALSE;

PROPERTY Running : BOOL READ
    Running := _running;
END_PROPERTY
```

This kind of structure (interfaces, methods, properties) is common in modern ST on Siemens/Beckhoff/CODESYS.

---

### Bottom line

- **Modern ST** is the mainstream “modern language” _on_ PLCs.
    
- If you need more power or already have C/C++ expertise, pick a platform that **lets you write realtime C/C++ modules** (e.g., TwinCAT 3, PLCnext) and keep orchestration/visualization in ST or .NET.
    
- For connectivity/analytics, use **Python/Rust/Go** at the **edge**, not in the tight control loop.
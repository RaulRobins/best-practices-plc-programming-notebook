# Project organization

Proper project organization is the foundation of maintainable, scalable, and collaborative PLC programming. A well-structured project not only enhances code readability but also significantly reduces debugging time, facilitates team collaboration, and ensures long-term maintainability of industrial automation systems.

This document outlines standardized approaches for organizing PLC projects across different platforms, focusing on creating consistent structures that can be easily understood by programmers, maintenance technicians, and engineers throughout the project lifecycle.

## 🔑 Key Benefits

- Improved Maintainability: Clear structure makes it easier to locate and modify code components.
- Enhanced Collaboration: Standardized organization enables seamless teamwork.
- Reduced Commissioning Time: Logical grouping of code accelerates debugging and testing.
- Scalability: Modular design allows for easy project expansion and modification.
- Knowledge Transfer: Consistent structure simplifies onboarding of new team members.

## Choosing the right language

| Use Case                                     | Recommended Language            | Notes                                                                                                                                                                                                                                                                                                                                    |
| -------------------------------------------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Boolean logic, interlocks, safety**        | **LAD / FBD**                   | Ladder (LAD) and Function Block Diagram (FBD) are excellent for logical conditions, permissives, and safety interlocks. They visually show relationships between bits and are immediately readable by electricians and maintenance personnel.                                                                                            |
| **Mathematical or structured logic**         | **SCL (Structured Text)**       | Structured Control Language (ST) offers clean syntax for math operations, comparisons, loops, and array manipulation. Use it for calculations, counters, or control algorithms where ladder becomes unreadable.                                                                                                                          |
| **Sequential machine control**               | **SFC (Graph)** or **SCL CASE** | Sequential Function Chart (Graph) is ideal for visualizing step-by-step sequences with transitions, especially when operators or process engineers need to follow logic visually. SCL using enumerated states (`CASE Step OF`) gives more compact, flexible control—better for advanced programmers or when steps need parameterization. |
| **Device drivers, motion blocks, actuators** | **FB in SCL**                   | Devices like valves, cylinders, motors, and axes often need memory for state tracking (e.g., “command active,” “done,” “faulted”). Implement these in **Function Blocks** written in **SCL** for reusability and clarity. Each instance keeps its own state, and the logic can be tested independently.                                  |
| **Simple combinational logic**               | **FC in LAD**                   | Use **Function Calls** in Ladder when logic is stateless (pure input → output). Great for small evaluations such as “if all safety doors are closed and e-stop reset, set SystemReady.” Keeps readability and prevents unnecessary DB overhead.                                                                                          |

## Variable Naming Convetions

### Variables

Use **lowerCamelCase** with type prefixes:

| Type                | Prefix | Example                      |
| ------------------- | ------ | ---------------------------- |
| BOOL                | b      | `bStartReq`, `bPermissiveOK` |
| INT/DINT            | i / di | `iCounter`, `diPartID`       |
| REAL                | r      | `rTorqueNm`, `rTempC`        |
| TIME                | t      | `tPulse100ms`                |
| STRING              | s      | `sSerial`                    |
| DATE_AND_TIME       | dt     | `dtStamp`                    |
| ARRAY               | arr    | `arrTorque[1..10]`           |
| STRUCT/UDT instance | st     | `stAxis1`, `stPerms`         |

- Constants or enums: `cMaxTorqueNm`, `EState`.
- Add **units** as suffix: `rSpeedRpm`, `tDelayMs`.

> Consistency > style. Choose one convention and keep it across all PLCs.

### 🧩 Blocks / Data Types

Use **PascalCase** with functional prefixes:

| Element          | Prefix | Example                             |
| ---------------- | ------ | ----------------------------------- |
| Function Block   | FB\_   | `FB_AxisControl`, `FB_ConveyorMain` |
| Function         | FC\_   | `FC_PermissiveOK`, `FC_ScaleAI`     |
| Data Block       | DB\_   | `DB_GlobalCfg`, `DB_IOMapping`      |
| User Data Type   | UDT\_  | `UDT_StationCmd`, `UDT_Permissives` |
| Sequential Chart | SFC\_  | `SFC_MainSequence`                  |

### When To Use FC VS FB

#### FC (Function Block)

- Requires **memory/state** (timers, latches, previous values).
- Used per **machine, axis, station**, or subsystem.
- Each FB has its own instance DB.

#### FB (Function Block)

- Requires **memoty/state** (timers, latches, previous values).
- Used per **machine, axis, station**, or subsystem.
- Each FB its own instance DB.

#### Instance DB Naming

> DB*[Equipment]*[Function]\_[Instance]

## 📂 Folder / block organization

```shell

ORGANIZATION BLOCKS (top-level)
 ├─ OB100 (startup)   - call FC_SelfTest, load defaults, init FBs
 ├─ OB1   (cyclic)    - call FB_SeqManager, FB_Logger, HMI comm
 ├─ OB35  (hardware interrupt / fast IO) - optional
 └─ FC_MainProgram

01_Sequence
 ├─ FB_ProductionSequence (FB)
 ├─ FB_ModeSequence (FB)
 ├─ FB_InitSequence (FB)
 └─ DB_SeqInstance (DB)

02_Defaults
 ├─ FC_Defaults (FC)
 └─ DB_Defaults (DB)

03_Functions
 ├─ Fastening
 │   ├─ FB_Fastening (FB)
 │   └─ DB_Fastening_Instance (DB)
 ├─ Riveting
 │   ├─ FB_Riveting (FB)
 │   └─ DB_Riveting_Instance (DB)
 ├─ Scanning
 │   ├─ FB_Scanning (FB)
 │   └─ DB_Scanning_Instance (DB)
 ├─ Printing
 │   ├─ FB_Printing (FB)
 │   └─ DB_Printing_Instance (DB)
 └─ Utilities
     ├─ FC_IO_Mapping (FC)
     └─ FC_TimerUtils (FC)

04_HMI
 ├─ FB_Alarms (FB)           - DB_FB_Alarms_Inst
 ├─ FB_Messages (FB)         - DB_FB_Messages_Inst
 ├─ FB_SeqStatus (FB)        - DB_FB_SeqStatus_Inst
 ├─ DB_HMI (DB)
 └─ FC_HMI_Comm (FC)

05_Counters
 ├─ FB_ProductionCounter
 └─ FB_DateTimeAndShifts

06_HMI_Remote
 ├─ FB_Alarms_Remote (FB)    - DB_FB_Alarms_Remote_Inst
 ├─ FB_Messages_Remote (FB)  - DB_FB_Messages_Remote_Inst
 ├─ FB_SeqStatus_Remote (FB) - DB_FB_SeqStatus_Remote_Inst
 ├─ DB_HMI_Remote (DB)
 └─ FC_HMI_Remote_Comm (FC)

07_Recipes
 ├─ DB_RecipeCurrent (DB)
 ├─ DB_RecipeEdit (DB)
 ├─ DB_RecipeList (DB)
 └─ FB_RecipeManager (FB)   - DB_FB_RecipeManager_Inst

08_Diagnostics
 ├─ FB_Logger (FB)           - DB_FB_Logger_Inst
 ├─ FC_SelfTest (FC)
 ├─ FC_ErrorHandler (FC)
 └─ DB_Log (DB)

09_Libraries
 ├─
 ├─
 ├─
 └─
´´´

```

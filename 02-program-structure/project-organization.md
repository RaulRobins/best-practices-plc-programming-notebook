# Project organization

Proper project organization is the foundation of maintainable, scalable, and collaborative PLC programming. A well-structured project not only enhances code readability but also significantly reduces debugging time, facilitates team collaboration, and ensures long-term maintainability of industrial automation systems.

This document outlines standardized approaches for organizing PLC projects across different platforms, focusing on creating consistent structures that can be easily understood by programmers, maintenance technicians, and engineers throughout the project lifecycle.

## Key Benefits

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



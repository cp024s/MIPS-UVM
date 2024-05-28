# DESIGN AND VERIFICATION OF MIPS ISA

## 1. Introduction to RV32I

The RV32I architecture is a subset of the RISC-V instruction set designed for 32-bit implementations. It serves as the foundation for a wide range of computing systems, from embedded devices to general-purpose computers.

### Key Features:
- RV32I is designed to be simple, efficient, and extensible, making it suitable for various applications.
- It provides a minimal set of instructions for essential arithmetic, logical, and control flow operations.
- RV32I is fully compatible with the RISC-V specification, ensuring interoperability and portability across different implementations.

---

## 2. Instruction Formats

RV32I instructions are encoded using several formats, each tailored to accommodate different types of operations and data.

### Formats Overview:
- **R-Type**: Used for register-register operations, including arithmetic and logical operations.
- **I-Type**: Enables operations with immediate values, such as loads, stores, and arithmetic/logical operations.
- **S-Type**: Facilitates store operations by specifying a memory address relative to a base register.
- **B-Type**: Handles conditional branch instructions based on specified conditions.
- **U-Type**: Supports instructions with an upper immediate value, allowing for efficient address calculation.
- **J-Type**: Facilitates jump instructions for altering the program flow.

### Instruction Set Summary

The RV32I instruction set includes instructions for arithmetic, logical, control flow, and memory operations. Here's a summary of the key instructions and their formats:

| Opcode | Type    | Description           | ALU | sel_A | sel_B | wb_sel | imm_gen | br_type | alu_op | reg_wr | d_wr |
|--------|---------|-----------------------|-----|-------|-------|--------|---------|---------|--------|--------|------|
| 3      | I-Type  | Load                  | yes | 1     | 1     | 2      | 0       | none    | 0      | 1      | 0    |
| 19     | I-Type  | Immediate Operation   | yes | 1     | 1     | 1      | 0       | none    | xxxx   | 1      | 0    |
| 23     | U-Type  | auipc                 | yes | 0     | 1     | 1      | 4(u)    | none    | 0      | 1      | 0    |
| 35     | S-Type  | Store                 | yes | 1     | 1     | z      | 1       | none    | 0      | 0      | 1    |
| 51     | R-Type  | Register Operation    | yes | 1     | 0     | 1      | z       | none    | xxxx   | 1      | 0    |
| 55     | U-Type  | lui                   | no  | 1     | 1     | 1      | 4(u)    | none    | copy   | 1      | 0    |
| 99     | B-Type  | Conditional Branch    | yes | 0     | 1     | z      | 2       | xxx     | 0      | 0      | 0    |
| 103    | I-Type  | jalr                  | yes | 1     | 1     | 0      | 0       | uncond  | 0      | 1      | 0    |
| 111    | J-Type  | jal                   | yes | 1     | 1     | 0      | 3       | uncond  | 0      | 1      | 0    |

### OP Code Split

|Hex values| Func 7 | RS2   | RS1   | Fun3 | rd   | opcode |Type        |
|---------|--------|-------|-------|------|------|--------|------------ |
| 04000413| 0000010| 00000 | 00000 | 000  | 01000| 0010011| opcode:19 
| 03200493| 0000001| 10010 | 00000 | 000  | 01001| 0010011| opcode:19
| 00801023| 0000000| 01000 | 00000 | 001  | 00000| 0100011| opcode:35 store
| 00901123| 0000000| 01001 | 00000 | 001  | 00010| 0100011| opcode:35 store
| 00001483| 0000000| 00000 | 00000 | 001  | 01001| 0000011| opcode:3 I type load
| 00201403| 0000000| 00010 | 00000 | 001  | 01000| 0000011| opcode:3 I type load
| 00005483| 0000000| 00000 | 00000 | 101  | 01001| 0000011| opcode:3 I type load
| 00005483| 0000000| 00000 | 00000 | 101  | 01001| 0000011| opcode:3 I type load
| 00940e63| 0000000| 01001 | 01000 | 000  | 11100| 1100011| opcode:99 B type conditional branch
| 00944663| 0000000| 01001 | 01000 | 100  | 01100| 1100011| opcode:99 B type conditional branch
| 40940433| 0100000| 01001 | 01000 | 000  | 01000| 0110011| opcode:51
| ff5ff06f| 1111111| 10101 | 11111 | 111  | 00000| 1101111| opcode:111
| 408484b3| 0100000| 01000 | 01001 | 000  | 01001| 0110011| opcode:51
| fedff06f| 1111111| 01101 | 11111 | 111  | 00000| 1101111| opcode:111
| 00900123| 0000000| 01001 | 00000 | 000  | 00010| 0100011| opcode:35
| 0000006f| 0000000| 00000 | 00000 | 000  | 00000| 1101111| opcode:111

### 2.1 R-Type Instructions
R-Type instructions in RV32I involve operations between two registers, typically used for arithmetic and logical operations.

### Components Breakdown:
- **Opcode**: Specifies the operation to be performed.
- **RS1, RS2**: Source registers for the operation.
- **RD**: Destination register for storing the result.
- **Funct3, Funct7**: Function fields providing additional operation details.

### 2.2 I-Type Instructions
- I-Type instructions include immediate values within the instruction itself, enabling operations with constant values.

### Usage Scenarios:
- Common operations include immediate arithmetic, loads, stores, and branch comparisons.

### 2.3 S-Type Instructions
- S-Type instructions facilitate storing data into memory, allowing the program to interact with the external memory system.

### Functional Overview:
- These instructions specify a memory address using a base register and an immediate offset.

### 2.4 B-Type Instructions
- B-Type instructions handle conditional branches, providing mechanisms for altering the program flow based on specified conditions.

### Conditional Branching:
- Branches are taken if the specified condition is met; otherwise, program execution continues linearly.

### 2.5 U-Type Instructions
- U-Type instructions offer extended immediate values for certain operations, enhancing the flexibility of instruction encoding.

### Example:
- The AUIPC instruction sets the upper 20 bits of the destination register to the immediate value.

### 2.6 J-Type Instructions
- J-Type instructions enable jump operations within the program, facilitating control flow management.

---

## 3. Design

### 3.1 Instruction Fetch Unit:
- Fetches instructions from memory using the program counter (PC).
- Utilizes the PC to access the instruction memory.
- Fetches the next instruction sequentially.
- Updates the PC to point to the next instruction.
- Interfaces with the memory subsystem for instruction retrieval.

### 3.2 Instruction Decode Unit:
- Decodes fetched instructions to determine their type and operands.
- Identifies the operation to be performed by the execution unit.
- Extracts necessary data fields from the instruction.
- Generates control signals based on the instruction opcode.
- Communicates decoded information to the execution unit.

### 3.3 Execution Unit:
- Performs arithmetic, logic, and shift operations on data.
- Utilizes ALU to execute arithmetic and logical instructions.
- Executes branch and jump instructions to alter control flow.
- Updates register values based on the operation result.
- Interfaces with the memory unit for load and store operations.

### 3.4 Memory Access Unit:
- Facilitates data transfers between the processor and memory.
- Handles load and store instructions to access memory.
- Manages memory addresses and data alignment.
- Coordinates data movement between registers and memory.
- Ensures proper synchronization and timing for memory access.

### 3.5 Register File:
- Stores general-purpose registers used by the processor.
- Provides fast access to register values for execution units.
- Facilitates data movement between registers and functional units.
- Supports read and write operations to update register values.
- Maintains register state during program execution.

### 3.6 ALU (Arithmetic Logic Unit):
- Performs arithmetic and logical operations on data operands.
- Executes operations such as addition, subtraction, AND, OR, etc.
- Supports shift and comparison operations.
- Generates result flags such as zero, carry, and overflow.
- Interfaces with the register file and control unit for operation execution.

### 3.7 Memory Interface:
- Acts as a bridge between the processor and external memory.
- Handles memory transactions such as reads and writes.
- Manages memory addresses and data transfer protocols.
- Ensures proper timing and synchronization for memory access.
- Interfaces with the memory controller to coordinate access.

### 3.8 Instruction Decoder:
- Decodes instruction opcodes to generate control signals.
- Determines the operation to be performed by the execution unit.
- Extracts necessary fields from the instruction encoding.
- Generates control signals for datapath units.
- Interfaces with the control unit to synchronize operations.

### 3.9 Control Unit:
- Coordinates the generation and propagation of control signals.
- Synchronizes the operation of datapath units.
- Responds to decoded instruction information.
- Controls the flow of data and instructions within the processor.
- Ensures proper sequencing and timing of operations.

### 3.10 Branch Control:
- Determines whether to take a branch based on conditions.
- Evaluates branch conditions and updates the program counter accordingly.
- Handles branch instructions such as conditional branches and jumps.
- Interfaces with the PC module to modify program flow.
- Manages branching logic and decision-making processes.

### 3.11 Multiplexer (Mux):
- Selects between two input signals based on a control signal.
- Routes data or control signals to the appropriate functional units.
- Acts as a switching element in the datapath.
- Supports conditional or unconditional selection of inputs.
- Facilitates data or control signal multiplexing within the processor.

### 3.12 Program Counter (PC):
- Stores the address of the next instruction to be fetched.
- Manages the program execution sequence.
- Increments sequentially for sequential instruction execution.
- Updates based on branching or control flow instructions.
- Interfaces with the instruction fetch unit to fetch instructions.

---

## 4. TEST BENCH

A testbench is a set of components, such as stimuli generators, monitors, and scoreboards, designed to verify the functionality of a design or module. It creates an environment where the design under test (DUT) can be subjected to various test scenarios to ensure its correctness and robustness.

UVM, or Universal Verification Methodology, is a comprehensive suite of standard tools and APIs tailored for verifying designs and Systems on Chip (SoCs). Developed atop the System Verilog language, UVM comprises essential class libraries crucial for building robust, reusable verification environments. Supported by simulators from various vendors, UVM streamlines the digital design process, enhancing productivity significantly. End users can perceive UVM as a versatile toolbox equipped with tools and guidelines essential for executing critical verification tasks effectively.

Here is a breakdown of each class:

### 4.1. Top Module
- **Clock and Reset Generation**: Generates clock and reset signals for the testbench.
- **Interface and Instance Setup**: Defines interfaces and instances required for communication.
- **Testbench Configuration**: Sets up interface configuration and runs the test.
- **Initialization**: Initializes clock, reset, and interfaces during the initial block.
- **Logging**: Logs testbench setup and execution details for debugging and analysis.

### 4.2. Interface
- **Signal Definition**: Defines input and output signals for communication.
- **Modport Definition**: Defines a modport to specify interface signals for DUT interaction.
- **Interface Connectivity**: Provides connectivity to the DUT for transaction exchange.
- **Data Handling**: Manages data exchange between the testbench and DUT.
- **Modular Design**: Promotes modularity by encapsulating interface functionality.

### 4.3. Sequence Item
- **Transaction Definition**: Defines transaction format including instruction and result fields.
- **Randomization**: Randomizes transaction fields within specified constraints.
- **Constraint Handling**: Defines constraints to restrict transaction field values.
- **Initialization**: Initializes sequence item properties during creation.
- **Modular Design**: Promotes modularity by encapsulating transaction properties.

### 4.4. Sequence
- **Transaction Generation**: Generates sequence items with randomized data for testing.
- **Item Start and Finish**: Starts and finishes sequence items for execution.
- **Randomization Control**: Controls randomization and generation of sequence items.
- **Logging**: Logs sequence item details for debugging and analysis.
- **Modular Design**: Promotes modularity by encapsulating sequence generation logic.

### 4.5. Sequencer
- **Sequence Item Management**: Manages the flow of sequence items to the driver.
- **Item Retrieval**: Retrieves sequence items from the sequence for execution.
- **Execution Control**: Controls the execution flow of sequence items.
- **Modular Design**: Promotes modularity by encapsulating sequencer functionality.
- **Initialization**: Initializes sequencer properties during creation.

### 4.6. Driver
- **Interface Connectivity**: Establishes a connection with the interface to drive transactions into the DUT.
- **Transaction Execution**: Executes transactions by driving instruction data into the interface.
- **Configuration Handling**: Retrieves interface configuration from the UVM configuration database during the build phase.
- **Continuous Operation**: Operates continuously to drive transactions into the DUT using a forever loop.
- **Logging**: Logs transaction details such as input data for debugging and analysis.

### 4.7. Monitor 1
- **Interface Connectivity**: Connects to the interface to monitor instruction data.
- **Transaction Observation**: Observes instruction data from the interface and forwards it to downstream components.
- **Data Forwarding**: Writes monitored data to an analysis port for downstream processing.
- **Continuous Monitoring**: Monitors instruction data continuously using a forever loop.
- **Logging**: Logs monitored data for debugging and analysis.

### 4.8. Monitor 2
- **Interface Connectivity**: Connects to the interface to monitor result data.
- **Transaction Observation**: Observes result data from the interface and forwards it to downstream components.
- **Data Forwarding**: Writes monitored data to an analysis port for downstream processing.
- **Continuous Monitoring**: Monitors result data continuously using a forever loop.
- **Logging**: Logs monitored data for debugging and analysis.

### 4.9. Agent 1
- **Component Composition**: Composes the sequencer, driver, and monitor components within the agent.
- **Sequencer-Driver Connection**: Connects the sequencer and driver components for transaction flow.
- **Monitor Interaction**: Interacts with monitor 1 to receive monitored data for analysis.
- **Initialization**: Initializes the components during the build phase.
- **Modular Design**: Adheres to a modular design approach to promote flexibility and scalability.

### 4.10. Scoreboard
- **FIFO Setup**: Sets up FIFOs for storing input and output transactions.
- **Transaction Processing**: Processes transactions from FIFOs for analysis.  
- **Instruction Type Identification**: Identifies the type of instruction based on opcode for analysis.
- **Continuous Operation**: Operates continuously to process transactions using a forever loop.
- **Logging**: Logs processed data and instruction types for debugging and analysis.

### 4.11. Agent 2
- **Component Composition**: Composes the monitor 2 and scoreboard components within the agent.
- **Monitor-Scoreboard Connection**: Connects monitor 2 to the scoreboard for data forwarding. 
- **Initialization**: Initializes the components during the build phase.
- **Modular Design**: Adheres to a modular design approach to promote flexibility and scalability.

### 4.12. Environment
- **Agent Composition**: Composes agent 1, agent 2, and scoreboard components within the environment.
- **Inter-Agent Connectivity**: Connects agents and scoreboard for communication and data exchange.
- **Initialization**: Initializes the components during the build phase.
- **Modular Design**: Adheres to a modular design approach to promote flexibility and scalability.

### 4.13. Test
- **Test Scenario Setup**: Sets up test scenarios by starting the sequence on agent 1.
- **Environment Setup**: Initializes the test environment by creating agents and the scoreboard.
- **Test Execution**: Executes the test scenarios by starting the sequence on agent 1.
- **Initialization**: Initializes the components during the build phase.
- **Logging**: Logs test execution details for debugging and analysis.

Got it! Here's a concise conclusion section you can add to your document:

---

## 5. Conclusion

In this document, we provided an overview of the RV32I instruction set architecture (ISA), highlighting its key features, instruction formats, and design components. The RV32I architecture serves as a foundational framework for building efficient and scalable computing systems, offering a minimalistic yet powerful set of instructions for a wide range of applications.

By understanding the fundamentals of the RV32I ISA and its associated design and verification techniques, engineers can develop reliable and efficient processor implementations for a variety of computing systems.

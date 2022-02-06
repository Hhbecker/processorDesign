# processorDesign

This repository contains my design of the microarchitecture for a simple 8 bit processor based on the [LC3] (https://github.com/chiragsakhuja/lc3tools).

### Specifications:
* The processor has 4 registers, each of 8 bits.
* All data types are 2’s complement integers.
* The processor has a RAM memory with 8-bit addressability (each location is 8 bits)
and address space of 256.
* RAM is used to store both data and instructions.
* The processor has 5 instructions


### Instruction Encodings/Opcodes:
1. Load


2. Store


3. Add


4. And


5. Branch


#### Register Encoding: 
R0 = 0
R1 = 1
R2 = 10
R3 = 11




### The Instruction Cycle:
1. **Fetch:** load instruction at address stored in PC into instruction register. Increment PC
2. **Decode:** Identify opcode. Idenntify operand location.
3. **Evaluate Address:** Add offset to compute operand address.
4. **Fetch Operands:** Fetch from addresses computed in previou step
5. **Execute:** Send operands to Arithmatic/Logic Unit
6. **Store:** Write result to destination register. 


### List of Control Signals:
**Gate.IR** (1 bit) Controls flow of Instruction Register contents onto Global Bus.
**Gate.PC** (1 bit) Controls flow of Program Counter contents onto Global Bus.
**Gate.ALU** (1 bit) Controls flow of Arithmetic Logic Unit results onto Global Bus.
**Gate.Reg** (1 bit) Controls flow of Register File at specified source register onto Global Bus.
**Gate.MDR** (1 bit) Controls flow of Memory Data Register contents onto Global Bus.

**LD.PC** (1 bit) Write Enable signal for Program Counter register. 
**LD.IR** (1 bit) Write Enable signal for Instruction Register. 
**LD.MDR** (1 bit) Write Enable signal for Memory Data Register.
**LD.MAR** (1 bit) Write Enable signal for Memory Address Register.
**LD.Reg** (1 bit) Write Enable signal for specified register within Register File. 

**PC.Mux** (1 bit) Determines whether the PC is incremented by one or the sum of PC+Offset is loaded into the PC register.
**MDR.Mux** (1 bit) Determines whether MDR is loaded with RAM contents or contents of the global bus.
**ALU.control** (1 bit) Determines whether the ALU outputs the result of an ADD or an AND.
**Write.RAM** (1 bit) Write Enable on RAM memory. 

**BR.control** (1 bit) TRUE/FALSE result of Branch logic coming from ALU back into the control unit.
**DR** (2 bits) Destination register within the Register File for the bus contents to be written into.
**SR1** (2 bits) Register contents to be sent to ALU or to be sent to bus.
**SR2** (2 bits) Register contents to be sent to ALU as second input.


## Executing a LOAD instruction:
Instruction: LOAD R0 1010
Equivalent Binary Instruction: [0][0][0][0][1][0][1][0]

CLARIFICATION: All control signals are OFF unless specified as ON by the bullet points in each step.

**Step 1.** 
The program counter is sent onto the bus and the MAR stores the contents of the bus. The program counter contains an address where the instruction [00001010] is stored in RAM.

Turn on Gate.PC control signal to send contents of Program Counter to global bus
Turn on LD.MAR to load bus contents into MAR

**Step 2.** 
The MDR is loaded with the contents of memory at the address specified by MAR. The memory content at that address is the 8 bit instruction [00001010]. The contents of the MDR are sent to the bus. The contents of the bus are sent into the Instruction Register. The PC is incremented.

Turn on LD.MDR to load MDR with contents of RAM at MAR address
Turn on GateMDR to send MDR contents to bus 
Turn on LD.IR to load bus contents into IR
Turn on LD.PC to increment PC (The control lines to the PC multiplexor select increment)

**Step 3.** 
Send opcode from IR to Control unit. No control signals needed.

**Step 4.**
Because the opcode specifies a load, the control unit will orchestrate the sending of the sign-extended last four bits of the IR contents to the bus. This is the address that will specify the RAM contents to be loaded into the Register File.

Turn on Gate.IR which sends sign extended address specified in instruction [1010] to bus
Turn on LD.MAR which loads bus contents into MAR

**Step 5.** 
The contents of RAM at the address now stored in the MAR will be loaded into the MDR. The MDR contents will be sent onto the bus. The bus contents will be loaded into the register file. The destination register bit is sent to the Register File from the Instruction register to ensure the bus contents from the MDR are written into the correct destination register. In the given instruction the destination register is 0 therefore the bit 0 will be sent to the Decoder in the register file to allow write enable on only Register 0. 

Turn on LD.MDR
Turn on Gate.MDR
Turn on LD.Reg
Destination Register bit sent to Register File Decoder

After this step, the control unit returns to the first step of the instruction cycle. The PC contents are sent to the MAR. The MDR is loaded with the next instruction and the instruction is sent to the IR. 




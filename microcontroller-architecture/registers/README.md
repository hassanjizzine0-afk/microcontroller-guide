

# 🧠  Registers

## 📌 What You Will Learn

This guide explains **register-level programming** — the lowest level of microcontroller control before assembly language. You will understand:
- What registers are and why they exist
- How registers differ from Flash and RAM
- How to write to registers directly on Arduino (AVR) and STM32 (ARM)
- Why register manipulation is faster than standard libraries
- All register types: General Purpose, Program Counter, Stack Pointer, Status Register, I/O Registers

---

## 🏗️ Memory Architecture of a Microcontroller

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              MICROCONTROLLER                                │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │                              CPU CORE                                   │ │
│  │                                                                         │ │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │  │
│  │   │    ALU      │  │     PC      │  │     SP      │  │    SREG     │ │  │
│  │   │ (Calculator)│  │ (Program    │  │  (Stack     │  │  (Status    │ │  │
│  │   │             │  │  Counter)   │  │  Pointer)   │  │  Register)  │ │  │
│  │   └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │  │
│  │                                                                         │ 
│  │   ┌────────────────────────────────────────────────────────────────┐    │ 
│  │   │                    GENERAL PURPOSE REGISTERS                   │    │
│  │   │                                                                │    │ 
│  │   │   AVR:  R0    R1    R2    R3    ...    R30    R31              │    │
│  │   │   ARM:  R0    R1    R2    R3    ...    R14    R15 (PC)         │    │ 
│  │   │                                                                │    │ 
│  │   │   ↑                                                            │    │ 
│  │   │   └── CPU works ONLY with these for calculations!              │    │ 
│  │   └────────────────────────────────────────────────────────────────┘    │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                       │
│                    ┌───────────────┼───────────────┐                       │
│                    ▼               ▼               ▼                        
│  ┌────────────────────────┐ ┌─────────────┐ ┌────────────────────────────┐ │
│  │     FLASH Memory       │ │  RAM Memory │ │      I/O Registers         │ │
│  │     (Program Code)     │ │ (Variables) │ │  (PORTB, DDRB, TIMER, ADC) │ │
│  │                        │ │             │ │                            │ │
│  │  Non-volatile          │ │  Volatile   │ │  Control peripherals:      │ │
│  │  Stores .bin/.hex      │ │  Variables  │ │  - Pins (input/output)     │ │
│  │  Slow (5-10 cycles)    │ │  Stack/Heap │ │  - Timers                  │ │
│  │                        │ │  (2-3 cycles│ │  - ADC, UART, SPI, I2C     │ │
│  └────────────────────────┘ └─────────────┘ └────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## ⚡ The Fundamental Rule

> **The CPU can ONLY perform operations on data that is in its general-purpose registers (R0, R1, ..., R31 for AVR; R0-R15 for ARM).**

It cannot directly add two numbers from RAM. It must first load them into registers.

### Example: Adding Two Numbers

| Step | Operation | Location | Explanation |
|------|-----------|----------|-------------|
| 1 | `LOAD R1, [0x2000]` | RAM → Register | Load value from RAM into R1 |
| 2 | `LOAD R2, [0x2004]` | RAM → Register | Load value from RAM into R2 |
| 3 | `ADD R3, R1, R2` | Registers only! | Add R1 + R2, store in R3 |
| 4 | `STORE [0x2008], R3` | Register → RAM | Save result back to RAM |

**No instruction can directly add two RAM addresses. The CPU cannot do it.**

---

## 🎯 Why Registers Exist (3 Reasons)

### 1. Speed

| Memory Type | Access Time | Location | Relative Speed |
|-------------|-------------|----------|----------------|
| **Registers** | **1 clock cycle** | Inside CPU | Fastest |
| RAM | 2-3 clock cycles | Outside CPU | 2-3x slower |
| Flash | 5-10 clock cycles | Outside CPU | 5-10x slower |

### 2. Instruction Set Architecture

Most microcontrollers (AVR, ARM, PIC) are **load-store architectures** (also called RISC).

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOAD-STORE ARCHITECTURE                      │
│                                                                 │
│   Only two instruction types can access memory:                 │
│                                                                 │
│   ┌─────────┐     ┌─────────┐     ┌─────────────────────────┐   │
│   │  LOAD   │     │  STORE  │     │  Arithmetic Instructions │  │
│   │         │     │         │     │  (ADD, SUB, MUL, AND,    │  │
│   │ RAM →   │     │ Register│     │   OR, XOR, CMP, etc.)    │  │
│   │ Register│     │ → RAM   │     │                         │   │
│   └─────────┘     └─────────┘     │  Work ONLY with registers│  │
│                                   └─────────────────────────┘   │
│                                                                 │
│   Example:                                                      │
│   LOAD R1, [100]    ; RAM → R1                                  │
│   LOAD R2, [200]    ; RAM → R2                                  │
│   ADD  R3, R1, R2   ; R1 + R2 → R3 (registers only!)            │
└─────────────────────────────────────────────────────────────────┘
```

### 3. Peripheral Control

You cannot control physical pins through RAM or Flash. Dedicated **I/O registers** map directly to hardware.

```
┌─────────────────────────────────────────────────────────────────┐
│                    WHY I/O REGISTERS EXIST                      │
│                                                                 │
│   Physical Pin PB5 ──┬──► Register DDRB (Direction Control)     │
│                      │                                          │
│                      ├──► Register PORTB (Output Control)       │
│                      │                                          │
│                      └──► Register PINB (Input Reading)         │
│                                                                 │
│   There is NO WAY to control the pin through RAM or Flash.      │
│   The ONLY path is through these special registers.             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📋 Complete Register Classification

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              REGISTERS                                        │
│                                                                               │
│  ┌────────────────────────────────────────────────────────────────────────┐   │
│  │                    GENERAL PURPOSE REGISTERS                            │  │
│  │                                                                            │
│  │   AVR:  R0, R1, R2, ..., R31 (32 registers, each 8-bit)                 │  │
│  │   ARM:  R0, R1, R2, ..., R15 (16 registers, each 32-bit)                │  │
│  │                                                                            │
│  │   Purpose: Temporary data storage, arithmetic operands, function returns │ │
│  └────────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
│  ┌────────────────────────────────────────────────────────────────────────┐   │
│  │                    SPECIAL PURPOSE REGISTERS                             │ │
│  │                                                                            │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │  │
│  │   │     PC      │  │     SP      │  │    SREG     │  │     X/Y/Z   │    │  │
│  │   │  Program    │  │   Stack     │  │   Status    │  │  (AVR only) │    │  │
│  │   │  Counter    │  │  Pointer    │  │  Register   │  │  Index Regs │    │  │
│  │   └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │  │
│  │                                                                            │ 
│  │   ARM also has: LR (Link Register), PSR (Program Status Register)        │ │
│  └────────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
│  ┌────────────────────────────────────────────────────────────────────────┐   │
│  │                         I/O REGISTERS                                    │ │
│  │                                                                            │
│  │   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌───────┐ │  │
│  │   │  DDRx   │ │  PORTx  │ │  PINx   │ │  TCCRn  │ │  OCRn   │ │ ADMUX │ │  │
│  │   │Direction│ │  Output │ │  Input  │ │ Timer   │ │ Compare │ │  ADC  │ │  │
│  │   │         │ │  Data   │ │  Data   │ │ Control │ │  Value  │ │       │ │  │
│  │   └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └───────┘ │  │
│  │                                                                            │ 
│  │   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                       │  │
│  │   │  UDR    │ │  UBRR   │ │  SPDR   │ │  TWDR   │                       │  │
│  │   │ UART    │ │  Baud   │ │  SPI    │ │  I2C    │                       │  │
│  │   │  Data   │ │  Rate   │ │  Data   │ │  Data   │                       │  │
│  │   └─────────┘ └─────────┘ └─────────┘ └─────────┘                       │  │
│  └────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Detailed Register Descriptions

### 1. General Purpose Registers (R0 - R31 / R0 - R15)

| Property | AVR (Arduino Uno) | ARM (STM32) |
|----------|-------------------|-------------|
| **Quantity** | 32 registers (R0-R31) | 16 registers (R0-R15) |
| **Size** | 8 bits each | 32 bits each |
| **Total storage** | 256 bits (32 bytes) | 512 bits (64 bytes) |
| **Special roles** | R0: Temporary<br>R1: Zero reg (optional)<br>R26-R31: X,Y,Z pointers | R13: Stack Pointer<br>R14: Link Register<br>R15: Program Counter |
| **Access time** | 1 clock cycle | 1 clock cycle |

**What they store:**
- Temporary calculation results
- Loop counters
- Function arguments and return values
- Memory addresses (pointer registers)

---

### 2. Program Counter (PC)

The **Program Counter** is the most important register for code execution. It tells the CPU which instruction to execute next.

| Property | Value |
|----------|-------|
| **What it stores** | Address of the NEXT instruction to fetch from Flash |
| **Size (AVR)** | 16-bit (addresses up to 64KB) |
| **Size (STM32)** | 32-bit (addresses up to 4GB) |
| **Initial value** | 0x0000 (reset vector) |
| **Auto-increment** | Yes, by instruction size (1-4 bytes) |
| **Can be modified** | Yes (by jump, call, branch, interrupt) |

**How the PC works:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROGRAM COUNTER IN ACTION                     │
│                                                                  │
│   Power ON                                                       │
│       │                                                          │
│       ▼                                                          │
│   PC = 0x0000                                                   │
│       │                                                          │
│       ▼                                                          │
│   Fetch instruction from Flash[0x0000]                         │
│       │                                                          │
│       ▼                                                          │
│   Execute instruction                                           │
│       │                                                          │
│       ▼                                                          │
│   PC = PC + instruction_size (e.g., 2 bytes) → PC = 0x0002      │
│       │                                                          │
│       ▼                                                          │
│   Fetch next instruction from Flash[0x0002]                    │
│       │                                                          │
│       ▼                                                          │
│   (loop continues...)                                           │
└─────────────────────────────────────────────────────────────────┘
```

**What can change the PC:**
- **Normal execution:** PC increments automatically
- **Jump instruction (JMP):** PC = target address
- **Branch instruction (BRNE):** PC += offset if condition true
- **Call instruction (CALL):** Push PC to stack, then PC = function address
- **Return instruction (RET):** Pop PC from stack
- **Interrupt:** PC = interrupt vector address

---

### 3. Stack Pointer (SP)

The **Stack Pointer** holds the address of the current **top of the stack**.

| Property | AVR | ARM (STM32) |
|----------|-----|-------------|
| **What it stores** | Address of stack top | Address of stack top |
| **Stack direction** | Grows **downward** (towards lower addresses) | Grows **upward** (towards higher addresses) |
| **Initial value** | RAM end (e.g., 0x08FF) | RAM start |
| **PUSH operation** | Decrement SP, then store | Store, then increment SP |
| **POP operation** | Load, then increment SP | Decrement SP, then load |

**What is stored on the stack:**
- Return addresses (when calling functions)
- Local variables (automatic storage)
- Register values saved before interrupts
- Function parameters (when registers are insufficient)

**Example (AVR):**
```asm
PUSH R1    ; SP-- → store R1 at SP
PUSH R2    ; SP-- → store R2 at SP
POP R2     ; Load from SP → SP++
POP R1     ; Load from SP → SP++
```

---

### 4. Status Register (SREG / PSR)

The **Status Register** contains **condition flags** that reflect the result of the last arithmetic or logical operation.

| Flag | Name | When is it set? | Use case |
|------|------|-----------------|----------|
| **Z (Zero)** | Zero flag | Result = 0 | Check if two numbers are equal |
| **C (Carry)** | Carry flag | Operation produced a carry/borrow | Multi-byte addition, shift operations |
| **N (Negative)** | Negative flag | Result has MSB = 1 (negative in signed math) | Signed comparisons |
| **V (Overflow)** | Overflow flag | Signed overflow occurred | Signed comparisons |
| **H (Half-Carry)** | Half-carry flag (AVR) | Carry from bit 3 to bit 4 | BCD arithmetic |
| **T (Transfer)** | Bit copy flag (AVR) | Stores bit for copy operations | Bit manipulation |
| **I (Interrupt)** | Global interrupt enable | Interrupts enabled | Enables/disables interrupts |
| **Q (Saturation)** | Saturation flag (ARM) | Saturation occurred | DSP operations |

**How flags work:**
```c
int a = 5;
int b = 5;
int c = a - b;  // result = 0 → Z flag = 1 (Zero flag set)

if (c == 0) {   // CPU checks Z flag
    // This code runs
}
```

**Reading status register (AVR):**
```c
#include <avr/io.h>

uint8_t status = SREG;  // Read status register
if (status & (1 << SREG_Z)) {
    // Zero flag was set
}
```

---

### 5. I/O Registers

I/O registers are **special registers that control hardware peripherals**. They are memory-mapped, meaning they appear as memory addresses.

#### GPIO Registers (AVR)

| Register | Full Name | Purpose | Example |
|----------|-----------|---------|---------|
| **DDRx** | Data Direction Register | Configure pin as INPUT (0) or OUTPUT (1) | `DDRB = 0x20` (PB5 output) |
| **PORTx** | Output Register | Set output pin HIGH (1) or LOW (0) | `PORTB = 0x20` (PB5 HIGH) |
| **PINx** | Input Register | Read input pin state | `if (PINB & 0x20)` |

#### GPIO Registers (STM32)

| Register | Full Name | Purpose | Bits |
|----------|-----------|---------|------|
| **MODER** | Mode Register | 00=input, 01=output, 10=alternate, 11=analog | 2 bits per pin |
| **OTYPER** | Output Type Register | 0=push-pull, 1=open-drain | 1 bit per pin |
| **OSPEEDR** | Output Speed Register | 00=low, 01=medium, 10=high, 11=very high | 2 bits per pin |
| **PUPDR** | Pull-up/Pull-down Register | 00=none, 01=pull-up, 10=pull-down, 11=reserved | 2 bits per pin |
| **ODR** | Output Data Register | Set output value (1=HIGH, 0=LOW) | 1 bit per pin |
| **IDR** | Input Data Register | Read input value | 1 bit per pin |
| **BSRR** | Bit Set/Reset Register | Set/reset bits in one operation | Atomic operation |

#### Timer Registers (AVR)

| Register | Purpose |
|----------|---------|
| **TCCRnA** | Timer/Counter Control Register A (PWM mode, compare output) |
| **TCCRnB** | Timer/Counter Control Register B (clock source, prescaler) |
| **TCNTn** | Timer/Counter Register (current count value) |
| **OCRnA** | Output Compare Register A (value to compare with TCNT) |
| **OCRnB** | Output Compare Register B |
| **TIMSKn** | Timer Interrupt Mask Register (enable interrupt) |
| **TIFRn** | Timer Interrupt Flag Register (pending interrupts) |

#### ADC Registers (AVR)

| Register | Purpose |
|----------|---------|
| **ADMUX** | ADC Multiplexer Selection Register (select channel, reference voltage) |
| **ADCSRA** | ADC Control and Status Register A (enable, start, prescaler) |
| **ADCL** | ADC Data Register Low (lower 8 bits of result) |
| **ADCH** | ADC Data Register High (upper 2-8 bits of result) |

#### Communication Registers (AVR)

| Peripheral | Register | Purpose |
|------------|----------|---------|
| **UART** | UDR | USART Data Register (transmit/receive data) |
| | UBRR | USART Baud Rate Register |
| | UCSRA | USART Control and Status Register A |
| **SPI** | SPDR | SPI Data Register |
| | SPCR | SPI Control Register |
| | SPSR | SPI Status Register |
| **I2C** | TWDR | TWI Data Register |
| | TWCR | TWI Control Register |
| | TWSR | TWI Status Register |

---

## 🔧 Practical Examples

### Example 1: Blinking an LED (AVR - Arduino Uno)

**Code without Registers (Arduino Standard):**
```cpp
void setup() {
    pinMode(13, OUTPUT);
}

void loop() {
    digitalWrite(13, HIGH);
    delay(1000);
    digitalWrite(13, LOW);
    delay(1000);
}
```

**Code WITH Registers (AVR):**
```cpp
void setup() {
    // Set PB5 (pin 13) as output
    // DDRB = Data Direction Register for Port B
    // (1 << 5) = binary 00100000 = bit 5 set to 1
    DDRB |= (1 << 5);   // Set bit 5 to 1 (output)
}

void loop() {
    // Turn LED on
    // PORTB = output register for Port B
    PORTB |= (1 << 5);   // Set bit 5 to 1 (HIGH)
    
    delay(1000);
    
    // Turn LED off
    PORTB &= ~(1 << 5);  // Set bit 5 to 0 (LOW)
    
    delay(1000);
}
```

### Example 2: Blinking an LED (STM32)

```cpp
int main() {
    // 1. Enable clock for GPIOB
    // RCC = Reset and Clock Control
    // AHB1ENR = Advanced High-performance Bus 1 Enable Register
    // (1 << 1) = enable clock for GPIOB
    RCC->AHB1ENR |= (1 << 1);
    
    // 2. Configure PB0 as output
    // MODER = Mode Register (2 bits per pin)
    // Clear bits 0 and 1: &= ~(3 << 0) where 3 = binary 11
    GPIOB->MODER &= ~(3 << (0 * 2));  // Clear 2 bits for pin 0
    GPIOB->MODER |= (1 << (0 * 2));   // Set to 01 (output)
    
    // 3. Main loop
    while(1) {
        // Set PB0 HIGH
        GPIOB->ODR |= (1 << 0);
        delay(1000);
        
        // Set PB0 LOW
        GPIOB->ODR &= ~(1 << 0);
        delay(1000);
    }
}
```

### Example 3: Reading a Button (AVR)

```cpp
void setup() {
    // Set PB0 as input (DDRB bit 0 = 0)
    DDRB &= ~(1 << 0);
    
    // Enable pull-up resistor on PB0 (PORTB bit 0 = 1)
    PORTB |= (1 << 0);
    
    // Set PB5 as output (for LED)
    DDRB |= (1 << 5);
}

void loop() {
    // Read button state from PINB register
    if (PINB & (1 << 0)) {
        // Button NOT pressed (pull-up makes it HIGH)
        PORTB |= (1 << 5);   // LED ON
    } else {
        // Button pressed (pulled LOW)
        PORTB &= ~(1 << 5);  // LED OFF
    }
}
```

### Example 4: Timer Interrupt (AVR)

```cpp
#include <avr/io.h>
#include <avr/interrupt.h>

int main() {
    // Configure timer 0 for CTC mode
    TCCR0A = (1 << WGM01);   // CTC mode (Clear Timer on Compare Match)
    TCCR0B = (1 << CS01) | (1 << CS00);  // Prescaler = 64
    
    // Set compare value for 1ms at 16MHz with prescaler 64
    // 16MHz / 64 = 250,000 Hz
    // 250,000 / 1000 = 250 cycles per ms
    OCR0A = 249;  // 0-249 = 250 counts = 1ms
    
    // Enable compare match interrupt
    TIMSK0 = (1 << OCIE0A);
    
    // Enable global interrupts
    sei();
    
    while(1) {
        // Main loop - runs while timer interrupts in background
    }
}

ISR(TIMER0_COMPA_vect) {
    // This code runs every 1ms
    static uint16_t counter = 0;
    
    if (++counter >= 1000) {
        counter = 0;
        // This runs every 1 second
        PORTB ^= (1 << 5);   // Toggle LED on PB5
    }
}
```

---

## 📊 Register Bit Manipulation Cheat Sheet

| Operation | AVR (8-bit) | STM32 (32-bit) | What it does |
|-----------|-------------|----------------|--------------|
| Set bit N | `REG |= (1 << N)` | `REG |= (1U << N)` | Change bit N to 1 |
| Clear bit N | `REG &= ~(1 << N)` | `REG &= ~(1U << N)` | Change bit N to 0 |
| Toggle bit N | `REG ^= (1 << N)` | `REG ^= (1U << N)` | Flip bit N |
| Read bit N | `(REG >> N) & 1` | `(REG >> N) & 1` | Get value of bit N (0 or 1) |
| Set multiple bits | `REG |= (mask)` | `REG |= (mask)` | Set all bits in mask |
| Clear multiple bits | `REG &= ~(mask)` | `REG &= ~(mask)` | Clear all bits in mask |
| Set bits in range | `REG |= ((1 << (END+1)) - (1 << START))` | Same | Set bits from START to END |
| Clear bits in range | `REG &= ~(((1 << (END+1)) - (1 << START)))` | Same | Clear bits from START to END |
| Write value to multiple bits | `REG = (REG & ~mask) | (value << shift)` | Same | Set multiple bits to specific value |

### Common Bit Masks (AVR)

| Value | Binary | Common name |
|-------|--------|-------------|
| `(1 << 0)` | `0b00000001` | Bit 0 |
| `(1 << 1)` | `0b00000010` | Bit 1 |
| `(1 << 2)` | `0b00000100` | Bit 2 |
| `(1 << 3)` | `0b00001000` | Bit 3 |
| `(1 << 4)` | `0b00010000` | Bit 4 |
| `(1 << 5)` | `0b00100000` | Bit 5 |
| `(1 << 6)` | `0b01000000` | Bit 6 |
| `(1 << 7)` | `0b10000000` | Bit 7 |
| `0x0F` | `0b00001111` | Lower nibble (bits 0-3) |
| `0xF0` | `0b11110000` | Upper nibble (bits 4-7) |
| `0xFF` | `0b11111111` | All 8 bits |

---

## 🔄 How a Single Line of C Code Translates to Registers

### Your C code:
```c
int x = 5;
int y = 3;
int z = x + y;
```

### What the compiler generates (AVR assembly):
```asm
; Load immediate 5 into register R1
LDI R1, 5

; Load immediate 3 into register R2
LDI R2, 3

; Add R1 and R2, store in R3
ADD R3, R1, R2

; At this point, R3 = 8
```

### What happens in hardware (cycle by cycle):

```
Cycle 1: CPU fetches LDI R1,5 from Flash → PC increments (0x0000 → 0x0002)
Cycle 2: CPU executes: 5 → R1
Cycle 3: CPU fetches LDI R2,3 from Flash → PC increments (0x0002 → 0x0004)
Cycle 4: CPU executes: 3 → R2
Cycle 5: CPU fetches ADD R3,R1,R2 from Flash → PC increments (0x0004 → 0x0006)
Cycle 6: ALU takes R1(5) + R2(3) = 8 → writes to R3
```

---

## 📈 Performance Comparison

| Operation | Arduino Function | Register Manipulation | Speed Gain |
|-----------|-----------------|----------------------|------------|
| Set pin HIGH | `digitalWrite()` - 60 cycles | `PORTB |= (1<<5)` - 1 cycle | **60x faster** |
| Read pin state | `digitalRead()` - 50 cycles | `PINB & (1<<5)` - 1 cycle | **50x faster** |
| Toggle pin | `digitalWrite()` + delay - 100 cycles | `PINB ^= (1<<5)` - 1 cycle | **100x faster** |
| 1kHz PWM (8-bit) | 10-15% CPU usage | 1-2% CPU usage | **5-10x more efficient** |
| Bit test in loop | `if (digitalRead(pin))` - 100 cycles | `if (PINB & (1<<5))` - 2 cycles | **50x faster** |

**Why is register manipulation so much faster?**

```
Arduino function (digitalWrite):
┌─────────────────────────────────────────────────────────────────┐
│  digitalWrite(13, HIGH)                                        │
│       │                                                         │
│       ▼                                                         │
│  Check if pin is valid (range check)                           │
│       │                                                         │
│       ▼                                                         │
│  Look up which PORT and bit corresponds to pin 13              │
│       │                                                         │
│       ▼                                                         │
│  Disable interrupts (to be safe)                               │
│       │                                                         │
│       ▼                                                         │
│  Write to PORT register                                        │
│       │                                                         │
│       ▼                                                         │
│  Re-enable interrupts                                          │
│       │                                                         │
│       ▼                                                         │
│  Return                                                        │
└─────────────────────────────────────────────────────────────────┘
Total: ~60 clock cycles

Register manipulation (PORTB |= (1<<5)):
┌─────────────────────────────────────────────────────────────────┐
│  PORTB |= (1<<5)                                               │
│       │                                                         │
│       ▼                                                         │
│  Read PORTB into temporary register                            │
│       │                                                         │
│       ▼                                                         │
│  OR with constant (1<<5)                                       │
│       │                                                         │
│       ▼                                                         │
│  Write back to PORTB                                           │
└─────────────────────────────────────────────────────────────────┘
Total: 1-3 clock cycles
```

---

## 🎯 Summary Table: Register vs Flash vs RAM

| Feature | Registers | Flash (Program Memory) | RAM (Data Memory) |
|---------|-----------|----------------------|-------------------|
| **What is stored?** | Temporary data, calculations, control flags | Your compiled code (.bin/.hex) | Variables, stack, heap |
| **Location** | **Inside CPU** | External (on-chip) | External (on-chip) |
| **Access time** | **1 clock cycle** | 5-10 clock cycles | 2-3 clock cycles |
| **Size (AVR)** | 32 x 8-bit (32 bytes total) | 32KB - 256KB | 2KB - 8KB |
| **Size (STM32)** | 16 x 32-bit (64 bytes total) | 64KB - 2MB | 20KB - 512KB |
| **Volatile?** | Yes (lost on power-off) | **No (permanent)** | Yes (lost on power-off) |
| **Can CPU compute directly?** | ✅ **YES** | ❌ NO (must load to registers first) | ❌ NO (must load to registers first) |
| **Controls peripherals?** | Yes (I/O registers) | No | No |
| **Write method** | Direct by CPU | Programmer (ST-LINK/bootloader) | Direct by CPU |
| **Address space** | Separate (or memory-mapped) | 0x0000 - 0x7FFF | 0x0100 - 0x08FF (AVR) |

---

## 🧠 Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    REGISTER QUICK REFERENCE CARD                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  GENERAL PURPOSE REGISTERS                                             │  │
│  │  ┌──────────────────────────────────────────────────────────────────┐  │  │
│  │  │ AVR: R0 R1 R2 ... R31 (8-bit each)                               │  │  │
│  │  │ ARM: R0 R1 R2 ... R15 (32-bit each)                              │  │  │
│  │  │ Purpose: Calculations, temporary data                            │  │  │
│  │  └──────────────────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  SPECIAL PURPOSE REGISTERS                                             │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │   │
│  │  │ PC           │ │ SP           │ │ SREG (AVR)   │ │ PSR (ARM)    │  │   │
│  │  │ Program      │ │ Stack        │ │ Status       │ │ Program      │  │   │
│  │  │ Counter      │ │ Pointer      │ │ Register     │ │ Status Reg   │  │   │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘  │   │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │   │
│  │  │ X (R26/R27)  │ │ Y (R28/R29)  │ │ Z (R30/R31)  │ │ LR (ARM)     │  │   │
│  │  │ Index Reg    │ │ Index Reg    │ │ Index Reg    │ │ Link Reg     │  │   │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘  │   │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  I/O REGISTERS (AVR)                                                   │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐    │   │
│  │  │ DDRx   │ │ PORTx  │ │ PINx   │ │ TCCRn  │ │ OCRn   │ │ ADCMUX │    │   │
│  │  │ Data   │ │ Output │ │ Input  │ │ Timer  │ │ Compare│ │  ADC   │    │   │
│  │  │ Dir    │ │ Data   │ │ Data   │ │ Ctrl   │ │ Value  │ │ Mux    │    │   │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ └────────┘    │   │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  STATUS REGISTER FLAGS (AVR SREG)                                      │  │
│  │  ┌───┬───┬───┬───┬───┬───┬───┬───┐                                    │   │
│  │  │ I │ T │ H │ S │ V │ N │ Z │ C │                                    │   │
│  │  │Int│Bit│Car│Sign│Over│Neg│Zero│Car│                                 │   │
│  │  │En │Copy│ry │ │flow│ative│ │ry │                                    │   │
│  │  └───┴───┴───┴───┴───┴───┴───┴───┘                                    │   │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  BIT OPERATIONS (AVR 8-bit)                                            │  │
│  │  ┌────────────────────────────────────────────────────────────────┐    │  │
│  │  │ Set bit N:   REG |= (1 << N)                                   │    │  │
│  │  │ Clear bit N: REG &= ~(1 << N)                                  │    │  │
│  │  │ Toggle bit N: REG ^= (1 << N)                                  │    │  │
│  │  │ Read bit N:   (REG >> N) & 1                                   │    │  │
│  │  └────────────────────────────────────────────────────────────────┘    │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  MEMORY HIERARCHY (Speed)                                              │  │
│  │                                                                        │  │
│  │  FASTEST                                                               │  │
│  │     ↓                                                                  │  │
│  │  ┌──────────────┐                                                      │  │
│  │  │  REGISTERS   │ ← 1 clock cycle                                      │  │
│  │  └──────────────┘                                                      │  │
│  │     ↓ (2-3x slower)                                                    │  │
│  │  ┌──────────────┐                                                      │  │
│  │  │     RAM      │ ← 2-3 clock cycles                                   │  │
│  │  └──────────────┘                                                      │  │
│  │     ↓ (5-10x slower)                                                   │  │
│  │  ┌──────────────┐                                                      │  │
│  │  │    FLASH     │ ← 5-10 clock cycles                                  │
│  │  └──────────────┘                                                      │  │
│  │     ↓                                                                  │  │
│  │  SLOWEST                                                               │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---






**Registers** are the CPU's **WORKSPACE**:
- **General-purpose registers** (R0-R31) hold **data** for calculations
- **I/O registers** (DDRB, PORTB, TCCR0) **control** the hardware
- **Program Counter (PC)** holds the **address** of the next instruction
- **Stack Pointer (SP)** holds the **address** of the stack top
- **Status Register (SREG)** holds **condition flags** from operations







### The Ultimate Analogy:

| Component | Analogy | What it does |
|-----------|---------|--------------|
| **FLASH** | 📚 Library | Stores books (your instructions) permanently |
| **RAM** | 📝 Desk | Stores notes (your variables) temporarily |
| **Registers** | 👐 Your hands | Do the actual work (calculations, control) |
| **PC (Program Counter)** | 👆 Bookmark | Tells you which book page to read next |
| **SP (Stack Pointer)** | 📌 Post-it | Marks where you left off in nested tasks |
| **SREG (Status)** | 🚦 Traffic lights | Tells you if result was zero, positive, etc. |

**Without registers, the CPU cannot:**
- Add two numbers
- Compare two values
- Make decisions (if statements)
- Control pins (LEDs, motors, sensors)
- Work with timers or ADC
- Communicate over UART, SPI, or I2C

**Everything in a microcontroller passes through registers.**

---

## 📚 References

- [ATmega328P Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf)
- [STM32 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0390-stm32f405415-stm32f407417-stm32f427437-and-stm32f429439-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [AVR Instruction Set Manual](https://ww1.microchip.com/downloads/en/DeviceDoc/AVR-Instruction-Set-Manual-DS40002198.pdf)
- [ARM Architecture Reference Manual](https://developer.arm.com/documentation/ddi0487/latest)

---

# FIFO-analysis
This project presents a parameterized Adaptive FIFO (First-In, First-Out) buffer designed in Verilog, along with a complete simulation testbench for functional and performance analysis.
# FIFO – First In First Out
A FIFO (First-In First-Out) buffer is a memory structure in which the first data written is the first data read. FIFO ensures data is processed in the same sequence in which it arrives. It is widely used in communication systems, data streaming, pipelining, DMA, video/audio processing, and clock-domain crossing.

<img width="654" height="229" alt="image" src="https://github.com/user-attachments/assets/87244aea-01c5-446c-a8c7-1c7bdf39c94e" />

---

## Types of FIFO

### ‣Synchronous FIFO:
Uses a single clock for both read and write operations.
Simpler design.
Suitable when producer and consumer operate on the same clock domain.

### ‣Asynchronous FIFO (Non-Synchronous FIFO):
Uses separate clocks for read and write sides.
Used when data crosses different clock domains.
Needs additional logic (Gray counters, synchronizers) for safe transfer.

---

## FIFO Depth:
FIFO depth represents how many data items the FIFO can store at a time.
Depth is determined based on:
1. Write and read frequency
2. Burst size
3. Gaps in data arrival
4. Idle cycles
5. Duty cycles

A well-sized FIFO ensures no overflow or underflow even under worst-case conditions.

### Overflow
Overflow happens when:
Write frequency > Read frequency
Data arrives faster than it is removed.
FIFO becomes Full → new incoming data is lost.

### Underflow
Underflow happens when:
Read frequency > Write frequency
Consumer reads faster than data is produced.
FIFO becomes Empty → invalid or stale data is read.

### Idle Cycle
An idle cycle is a clock cycle in which no read or write operation happens.
Idle cycles occur due to:

Bus delays
Gaps between two bursts
Control logic wait states
Idle cycles reduce the effective bandwidth.

###Duty Cycle (Enable Duty)
Duty cycle defines how long the write/read enable stays HIGH within one clock period.

Example:
Write duty cycle = 50% → write enabled for half of the cycles
Read duty cycle = 25% → read enabled for one-fourth of cycles
Duty cycle affects the actual throughput, not just the frequency.

---

# Explanation of All 5 FIFO Cases (Based on Notes + Waveforms)

## Case-1

Write frequency = 100 MHz
Read frequency = 50 MHz
Burst size = 120
Observation

<img width="1519" height="697" alt="Screenshot 2025-07-16 204654" src="https://github.com/user-attachments/assets/8b8b0a5c-e8dd-4ed6-8f20-6c52a605868a" />

Write is twice as fast as read.
FIFO fills quickly → almost_full = 1
Read removes only half the data in the same time.
FIFO reaches near full, risk of overflow.

Depth Calculation:
- Time to write burst: 120 × 10 ns = 1200 ns
- Data read in 1200 ns: 1200 / 20 ns = 60 items
- Data to be stored = 120 – 60 = 60 items
- FIFO depth ≥ 60

---

## Case-2
Write = 200 MHz
Read = 50 MHz
Burst size = 120
Idle cycles:
1 idle cycle between writes
2 idle cycles between reads
Observation

<img width="1515" height="699" alt="Screenshot 2025-07-16 205004" src="https://github.com/user-attachments/assets/a0483006-8d2a-4fea-97b0-0f41bdb19a96" />

Even with idle cycles, write rate > read rate.
FIFO fills, but slower than Case-1 because write is not continuous.
Read has even more idle time → occupancy grows further.

Depth Calculation:
- Write time per data = 2 × (1/200 MHz) = 10 ns
- Read time per data = 3 × (1/50 MHz) = 60 ns
- Data read in 1200 ns = 1200/60 = 20 items
- Data stored = 120 – 20 = 100 items
- FIFO depth ≥ 100

---

## Case-3
Write = 200 MHz, write enable = 50% duty
Read = 50 MHz, read enable = 25% duty
Burst size = 120
Observation

<img width="1288" height="717" alt="Screenshot 2025-07-16 205320" src="https://github.com/user-attachments/assets/8438248e-82eb-43b1-a6fd-accb82599850" />

Write operates half the time → effective rate = 100 MHz
Read operates only 25% → effective rate = 12.5 MHz
FIFO fills very fast
full asserts frequently
Highest peak_usage among all cases.

Depth Calculation:
- Effective write time = 1/100 MHz = 10 ns
- Effective read time = 1/(50 MHz × 0.25) = 80 ns
- Data read in 1200 ns ≈ 15 items
- Data stored = 120 – 15 = 105 items
- FIFO depth ≥ 105

---

## Case-4
Write = 40 MHz
Read = 80 MHz
Burst = 120
Observation

<img width="1287" height="717" alt="Screenshot 2025-07-16 205451" src="https://github.com/user-attachments/assets/475f609b-f021-4961-9147-613d2f68a425" />

Read is twice as fast as write.
FIFO quickly empties → underflow risk.
empty and almost_empty stay HIGH most of the time.

Depth Calculation:
- In 3000 ns:
- Read can consume 240 items
- Write produces only 120
- Since read > write, FIFO depth requirement → very small
- FIFO not stressed in this case.

---

## Case-5
Write = 50 MHz
Read = 50 MHz
Burst Size = 120
Observation

<img width="1291" height="714" alt="Screenshot 2025-07-16 205558" src="https://github.com/user-attachments/assets/330bcb69-be76-45b9-b3c6-66961987de30" />

- Write and read speeds are equal.
- FIFO reaches a stable equilibrium.
- full and empty rarely occur.
- Ideal balanced streaming case.

---

### Conclusion:
FIFO is not required, or a very small depth is sufficient.

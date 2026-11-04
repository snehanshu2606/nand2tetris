# Hack HDL – Nand2Tetris Projects 1 & 2

## How to run
1. Download the official **nand2tetris software suite** (nand2tetris.org) and open the **Hardware Simulator**.
2. Copy the `.hdl` files from this folder into the project folders (`projects/1`, `projects/2`), next to the official `.tst` / `.cmp` files.
3. Load each `.hdl` (or its `.tst`), press **Run**, and confirm "End of script – Comparison ended successfully".
4. Take your screenshots from the simulator (I cannot run the GUI simulator here, so screenshots/output are yours to capture).

Files — Project 1: `Not And Or Xor Mux DMux Not16 And16 Mux16 Mux4Way16 Or8Way` · Project 2: `HalfAdder FullAdder Add16 ALU`
(`Mux4Way16` is the standard 4-way mux; for a 1-bit version use `Mux16` replaced by `Mux` in the same structure.)

---
## 1. Gates from NAND only
| Gate | Boolean expression | NAND construction |
|---|---|---|
| NOT | ¬a = a ↑ a | 1 NAND, inputs tied |
| AND | a·b = ¬(a ↑ b) = (a↑b)↑(a↑b) | 2 NANDs |
| OR | a+b = ¬(¬a·¬b) = (a↑a)↑(b↑b) | 3 NANDs (De Morgan) |

```
NOT:  in ──┬─┐
           └─┤NAND├── out

AND:  a ─┐            ┌─┐
         ├NAND─ n ──┬─┤NAND├── out
      b ─┘          └─┘

OR:   a ─┬─┐
         └─┤NAND├─ ¬a ─┐
                        ├NAND── out
      b ─┬─┐            │
         └─┤NAND├─ ¬b ─┘
```
Truth tables: NOT 0→1, 1→0. AND = 1 only for 11. OR = 0 only for 00.

## 2. XOR, Mux, DMux
**XOR** = a·¬b + ¬a·b

| a b | XOR |   | sel a b | Mux |   | in sel | a b |
|---|---|---|---|---|---|---|---|
| 0 0 | 0 |   | 0 0 0 | 0 |   | 0 0 | 0 0 |
| 0 1 | 1 |   | 0 0 1 | 0 |   | 0 1 | 0 0 |
| 1 0 | 1 |   | 0 1 0 | 1 |   | 1 0 | 1 0 |
| 1 1 | 0 |   | 0 1 1 | 1 |   | 1 1 | 0 1 |
|     |   |   | 1 0 0 | 0 |
|     |   |   | 1 0 1 | 1 |
|     |   |   | 1 1 0 | 0 |
|     |   |   | 1 1 1 | 1 |

Mux = a·¬sel + b·sel. DMux: a = in·¬sel, b = in·sel.

**Mux4Way16:** sel=00→a, 01→b, 10→c, 11→d (two Mux16 on sel[0], one on sel[1]).

## 3. Half Adder, Full Adder, carry trace
Half adder: sum = a⊕b, carry = a·b. Full adder: sum = a⊕b⊕c, carry = a·b + c·(a⊕b).

Trace a=1, b=1, c=1:
1. HA#1(1,1): s1 = 0, c1 = 1
2. HA#2(s1=0, c=1): sum = 1, c2 = 0
3. carry = c1 OR c2 = 1  → **sum=1, carry=1** (1+1+1 = 11₂ ✔)

## 4. Add16 (ripple carry) – sample runs
Bit 0 is a half adder; bits 1–14 full adders; bit 15's carry-out is discarded.

| a | b | out |
|---|---|---|
| 0000000000000101 (5) | 0000000000000011 (3) | 0000000000001000 (8) |
| 0101010101010101 | 0011001100110011 | 1000100010001000 |
| 0111111111111111 (+32767) | 0000000000000001 (+1) | 1000000000000000 (−32768) ← overflow |
| 1111111111111111 (−1) | 0000000000000001 (+1) | 0000000000000000 (0, carry discarded) |

Carry ripple for 5+3: bit0 1+1→sum 0,c=1; bit1 0+1+1→sum 0,c=1; bit2 1+0+1→sum 0,c=1; bit3 0+0+1→sum 1,c=0; rest 0 → `1000`.

**Two's complement:** a 16-bit word represents values −2¹⁵…2¹⁵−1. MSB has weight −32768; negate by inverting all bits and adding 1 (−x = ¬x + 1). One adder therefore handles signed and unsigned addition.

**Overflow:** the result wraps modulo 2¹⁶. Overflow (true result out of range) occurs when two operands of the same sign give a result of the opposite sign (equivalently carry-in to bit 15 ≠ carry-out of bit 15). Hack's Add16 doesn't flag it; it just wraps (32767+1 = −32768). Adding operands of opposite signs can never overflow.

## 5. ALU function table (18 operations)
| zx | nx | zy | ny | f | no | out |
|---|---|---|---|---|---|---|
| 1 | 0 | 1 | 0 | 1 | 0 | 0 |
| 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 1 | 1 | 1 | 0 | 1 | 0 | −1 |
| 0 | 0 | 1 | 1 | 0 | 0 | x |
| 1 | 1 | 0 | 0 | 0 | 0 | y |
| 0 | 0 | 1 | 1 | 0 | 1 | !x |
| 1 | 1 | 0 | 0 | 0 | 1 | !y |
| 0 | 0 | 1 | 1 | 1 | 1 | −x |
| 1 | 1 | 0 | 0 | 1 | 1 | −y |
| 0 | 1 | 1 | 1 | 1 | 1 | x+1 |
| 1 | 1 | 0 | 1 | 1 | 1 | y+1 |
| 0 | 0 | 1 | 1 | 1 | 0 | x−1 |
| 1 | 1 | 0 | 0 | 1 | 0 | y−1 |
| 0 | 0 | 0 | 0 | 1 | 0 | x+y |
| 0 | 1 | 0 | 0 | 1 | 1 | x−y |
| 0 | 0 | 0 | 1 | 1 | 1 | y−x |
| 0 | 0 | 0 | 0 | 0 | 0 | x&y |
| 0 | 1 | 0 | 1 | 0 | 1 | x\|y |

Flags: zr = 1 iff out = 0; ng = 1 iff out[15] = 1. Verify with `ALU.tst` / `ALU.cmp` (and `ALU-nostat.tst` first if you want to debug without flags).

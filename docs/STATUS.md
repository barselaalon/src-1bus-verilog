# STATUS

_Generated from the SRC 1-Bus tracker on 2026-10-26._

## 0. Kickoff

- [x] פתיחת repo ב-GitHub והגנה על main (Both, done)
- [x] docs/signals.md (החוזה בינינו, נכתב ראשון) (Both, done)
- [ ] docs/STATUS.md (ייצוא ראשון מהדף) (Both, todo)
- [x] סביבת עבודה: iverilog, Verilator, Surfer, VS Code, git (Alon, done)
- [ ] סביבת עבודה: iverilog, Verilator, Surfer, VS Code, git (Eran, todo)

## 1. Learn Verilog

- [ ] HDLBits: בסיס Verilog, צירופי וסדרתי (Alon, todo)
- [ ] HDLBits: בסיס Verilog, צירופי וסדרתי (Eran, todo)

## 2. Modules + cross testbenches + tools

- [ ] regfile.v (Gra/Grb/Grc, BAout) (Alon, todo) | testbench: Eran, todo
- [ ] ir.v (c1, c2 עם הרחבת סימן) (Alon, todo) | testbench: Eran, todo
- [ ] alu.v + רגיסטרים A ו-C (Alon, todo) | testbench: Eran, todo
- [ ] shift_counter.v (n, Ld, Decr, n=0) (Alon, todo) | testbench: Eran, todo
- [ ] con_logic.v (IR<2..0>, CONin, CON) (Alon, todo) | testbench: Eran, todo
- [ ] bus.v (mux one-hot לפי אותות out) (Alon, todo) | testbench: Eran, todo
- [ ] control_unit.v (מונה צעדים, מפענח, End) (Eran, todo) | testbench: Alon, todo
- [ ] mem_if.v (MA, MD, MDbus, MDrd) (Eran, todo) | testbench: Alon, todo
- [ ] memory.v ($readmemh, big-endian) (Eran, todo) | testbench: Alon, todo
- [ ] משוואות אותות: fetch ופקודות שלב א' (Eran, todo)
- [ ] tools/asm.py (אסמבלר SRC ל-hex) (Alon, todo)
- [ ] tools/iss.py (מודל ייחוס) (Eran, todo)

## 3. Integration until fetch works

- [ ] datapath.v (Both, todo)
- [ ] src_top.v (Both, todo)
- [ ] bus contention checker ב-testbench (Both, todo)
- [ ] fetch עובד: IR ו-PC נכונים אחרי T2 (Both, todo)

## 4. Stage A instructions + test programs

- [ ] מקרי קצה: ld/st עם rb=0 ו-rb≠0, c2 שלילי, כתיבה ל-R0, stop (Both, todo)
- [ ] assertion: מספר צעדים לכל פקודה מול עמ' 16 (Both, todo)
- [ ] תוכנית מתרגול 3: הגדול מבין שניים לכתובת 100 (Both, todo)
- [ ] תוכנית: סכום מערך בלולאה (Both, todo)
- [ ] השוואת רגיסטרים וזיכרון מול iss.py (Both, todo)

| Instr | Opcode | Status |
|---|---|---|
| nop | 0 | todo |
| ld | 1 | todo |
| st | 3 | todo |
| la | 5 | todo |
| br / brzr / brnz / brpl / brmi | 8 | todo |
| add | 12 | todo |
| addi | 13 | todo |
| sub | 14 | todo |
| and | 20 | todo |
| or | 22 | todo |
| not | 24 | todo |
| stop | 31 | todo |

## 5. README & wrap-up

- [ ] README.md (מה זה SRC, הפניה ל-Heuring & Jordan, איך מריצים) (Both, todo)
- [ ] יומן באגים ב-README (Both, todo)

## 6. Stage B (optional)

- [ ] latency קבועה בזיכרון ובדיקה ש-Wait עוצר (Both, todo)

| Instr | Opcode | Status |
|---|---|---|
| ldr | 2 | todo |
| str | 4 | todo |
| lar | 6 | todo |
| brl | 9 | todo |
| neg | 15 | todo |
| andi | 21 | todo |
| ori | 23 | todo |
| shr | 26 | todo |
| shra | 27 | todo |
| shl | 28 | todo |
| shc | 29 | todo |

## Decisions

- 2026-10-16 (Alon): צופה גלים: Surfer במקום GTKWave (ה-cask של GTKWave ב-Homebrew חסום מאוקטובר 2025).
- 2026-10-16 (Alon): הדשבורד הוא מקור האמת למעקב. docs/STATUS.md נוצר ממנו בייצוא.
- 2026-10-16 (Alon): אין לוח זמנים קשיח. מתקדמים לפי סדר השלבים, ושלב ב' רק אחרי ששלב א' עובר במלואו.

## Bugs

- (none yet)
# ERRANT LFIS

<p align="center">
  <img src="https://img.shields.io/badge/language-C_%2B_Haskell_%2B_Prolog_%2B_JS-2e86c1?style=flat-square"/>
  <img src="https://img.shields.io/badge/types-QTT_linear-c0392b?style=flat-square"/>
  <img src="https://img.shields.io/badge/audit-WORM_sealed-00b894?style=flat-square"/>
  <img src="https://img.shields.io/badge/datalog-8_stage_pipeline-8e44ad?style=flat-square"/>
  <img src="https://img.shields.io/badge/cert-METATRON-gold?style=flat-square"/>
  <img src="https://img.shields.io/badge/license-SSL_v3.0-555?style=flat-square"/>
</p>

<p align="center">
  <strong>Forth is the metal. Prolog is the law. Linear types are the vow. WORM is the memory. Ω</strong>
</p>

---

## What It Is

ERRANT is a **Linear Forth Instruction Set** — a stack machine where every value carries an ownership mode (QTT multiplicity), and the type system enforces that:

- **linear** values are consumed exactly once
- **affine** values are consumed at most once
- **unrestricted** values may be freely copied
- **capability** tokens are always linear and named
- **sealed** values are WORM artifacts — immutable, linear, final

The build either succeeds or it fails. There is no "almost compiled." There is no "mostly sovereign." METATRON does not certify partial agreement.

---

## Architecture

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                      ERRANT LFIS STACK                          │
  ├──────────────────┬──────────────────────────────────────────────┤
  │  errant.h / .c   │  C runtime — QTT multiplicity tracking,      │
  │  (runtime/)      │  opcode execution, SHA-256, WORM sealing,     │
  │                  │  FFI target for Node/Python/OCaml/Haskell     │
  ├──────────────────┼──────────────────────────────────────────────┤
  │  opcodes.mjs     │  JS opcode table — MODES + OPCODES            │
  │  interpreter.mjs │  JS VM — runErrant, disassemble, createImage  │
  │  test.mjs        │  JS test harness                              │
  ├──────────────────┼──────────────────────────────────────────────┤
  │  typing.pl       │  Prolog typing kernel                         │
  │  (Prolog)        │  type_op rules → valid_errant_image/2         │
  │                  │  DUP only on un, DROP only on aff,            │
  │                  │  SEAL requires cap(seal), HALT requires sealed │
  ├──────────────────┼──────────────────────────────────────────────┤
  │  datalog/        │  Haskell Datalog pipeline (8 stages)          │
  │  DatalogEngine.hs│  artifact → json_admissible → nfc_ok →        │
  │  Main.hs         │  snap_canonical → sha256_digest →             │
  │  errant-datalog  │  snap_address → accepted → worm_receipt       │
  │  .cabal          │                                               │
  ├──────────────────┼──────────────────────────────────────────────┤
  │  soul-spec.errant│  Soul spec — 49th Call, Al-Hamid, 231 gates,  │
  │                  │  METATRON certification, QTT capabilities      │
  ├──────────────────┼──────────────────────────────────────────────┤
  │  llm/            │  LLM integration via ERRANT linear types      │
  │  errant-ggml.hs  │  GGML bridge in Haskell                       │
  │  runtime.mjs     │  JS LLM runtime                               │
  │  worm-seal.mjs   │  WORM seal for LLM outputs                    │
  │  tensor-flow.pl  │  Prolog tensor flow rules                     │
  │  tensor-ops.wit  │  WIT interface definitions                    │
  └──────────────────┴──────────────────────────────────────────────┘
```

---

## QTT Value Modes

QTT = Quantitative Type Theory. Every value on the ERRANT stack carries a multiplicity:

```
  ╔════════════╦════════╦═══════════════════════════════════════════╗
  ║ Mode       ║ Symbol ║ Rule                                      ║
  ╠════════════╬════════╬═══════════════════════════════════════════╣
  ║ lin        ║  ⊸     ║ Must be consumed exactly once             ║
  ║            ║        ║ DUP → type error. DROP → type error.      ║
  ╠════════════╬════════╬═══════════════════════════════════════════╣
  ║ aff        ║  ⊸?    ║ May be consumed at most once              ║
  ║            ║        ║ DROP allowed. DUP → type error.           ║
  ╠════════════╬════════╬═══════════════════════════════════════════╣
  ║ un         ║  ∞     ║ Unrestricted — copy and drop freely       ║
  ║            ║        ║ DUP allowed. DROP allowed.                ║
  ╠════════════╬════════╬═══════════════════════════════════════════╣
  ║ cap        ║  ⚿     ║ Authority token — always linear           ║
  ║            ║        ║ Named. Cannot be forged. One use.         ║
  ╠════════════╬════════╬═══════════════════════════════════════════╣
  ║ seal       ║  🔒    ║ WORM artifact — sealed, immutable, linear  ║
  ║            ║        ║ HALT requires sealed(final) on stack.     ║
  ╚════════════╩════════╩═══════════════════════════════════════════╝
```

---

## Opcode Table

### Stack Operations

| Hex | Name | Stack Effect | Notes |
|-----|------|--------------|-------|
| `0x00` | NOP | `( -- )` | |
| `0x01` | PUSH_UN | `( -- un a )` | unrestricted push |
| `0x02` | PUSH_LIN | `( -- lin a )` | linear push |
| `0x03` | MOVE | `( lin a -- lin a )` | transfer ownership |
| `0x04` | DROP_AFF | `( aff a -- )` | drop affine only |
| `0x05` | DUP_UN | `( un a -- un a ⊗ un a )` | unrestricted only |
| `0x06` | SWAP | `( lin a ⊗ lin b -- lin b ⊗ lin a )` | |
| `0x07` | PAIR | `( lin a ⊗ lin b -- lin (a,b) )` | |
| `0x08` | UNPAIR | `( lin (a,b) -- lin a ⊗ lin b )` | |

### Control Flow

| Hex | Name | Stack Effect | Notes |
|-----|------|--------------|-------|
| `0x09` | APPLY | `( lin (a⊸b) ⊗ lin a -- lin b )` | linear function application |
| `0x0A` | CALL | `( lin frame -- lin result )` | |
| `0x0B` | RETURN | `( lin result -- )` | |
| `0x0C` | BRANCH | `( lin bool ⊗ lin l ⊗ lin r -- lin result )` | |
| `0x0D` | FAIL | `( lin error -- )` | linear error |

### Cryptographic Operations

| Hex | Name | Stack Effect | Notes |
|-----|------|--------------|-------|
| `0x10` | SEAL | `( lin cap(seal) ⊗ lin data -- lin sealed(data) )` | WORM seal |
| `0x11` | ATTEST | `( lin cap(attest) ⊗ lin sealed -- lin attest )` | |
| `0x12` | HASH | `( lin data -- lin hash )` | SHA-256 |
| `0x13` | SIGN | `( lin cap(sign) ⊗ lin hash -- lin sig )` | Ed25519 |
| `0x14` | VERIFY | `( un pk ⊗ lin sig ⊗ lin hash -- lin bool )` | |

### Agent Operations

| Hex | Name | Stack Effect | Notes |
|-----|------|--------------|-------|
| `0x20` | SPAWN | `( lin cap(spawn) ⊗ lin spec -- lin pid )` | |
| `0x21` | SEND | `( lin cap(send) ⊗ lin pid ⊗ lin msg -- lin rcpt )` | |
| `0x22` | RECV | `( lin cap(recv) ⊗ lin chan -- lin msg )` | |
| `0x23` | HALT | `( lin sealed(final) -- Ω )` | **must be last** |

### MAGMACORE Bridge Ops

| Hex | Name | Description |
|-----|------|-------------|
| `0x30` | FUSE | ADD — a+b |
| `0x31` | CUT | SUB — b-a |
| `0x32` | FORGE_OP | MUL — a*b |
| `0x33` | ECHO | output char |
| `0x34` | LISTEN | input char |
| `0x35` | GATE | jump to nearest mine |
| `0x36` | RUPTURE | mine detonation — MAGMA SENTINEL |
| `0x37` | ORIGIN | excavation start |
| `0x38` | RESONANCE | φ amplify — MAGMA ORACLE |
| `0x39` | TRANSFORM | stack rotate (a b c → b c a) |
| `0x3A` | MEMORY | store in cell — MAGMA ANCHOR |
| `0x3B` | REDUCTION | sum collapse — MAGMA VAULT |
| `0x3C` | PORTAL | teleport |

---

## Forbidden Operations

```
  DUP  on lin or cap    → TYPE ERROR: cannot copy linear resource
  DROP on lin or cap    → TYPE ERROR: linear resource leaked
  SEAL without cap(seal) → TYPE ERROR: no seal authority
  HALT with unconsumed lin on stack → TYPE ERROR: resource leak
  EXECUTE unverified frame → TYPE ERROR: no execution authority
```

---

## Prolog Typing Kernel

`typing.pl` is the ground truth for what programs are legal:

```prolog
% Only unrestricted values may be duplicated
type_op(dup_un, [item(un, A) | S], [item(un, A), item(un, A) | S]).

% Only affine values may be dropped
type_op(drop_aff, [item(aff, A) | S], S).

% SEAL requires the seal capability on top
type_op(seal,
  [item(lin, cap(seal)), item(lin, Data) | S],
  [item(lin, sealed(Data)) | S]).

% HALT requires a sealed final artifact — nothing else accepted
type_op(halt, [item(lin, sealed(final))], omega).

% Genesis invariant: valid program reaches omega
valid_errant_image(Program) :-
    check_program(Program, [], omega).
```

---

## Datalog Pipeline (Haskell)

`datalog/DatalogEngine.hs` implements an 8-stage non-recursive Datalog evaluator:

```
  Stage 1  artifact(A)              — input declared as artifact
  Stage 2  json_admissible(A)       — valid JSON format check
  Stage 3  nfc_ok(A, N)             — Unicode NFC normalization
  Stage 4  snap_canonical(N, B)     — canonical byte form
  Stage 5  sha256_digest(B, D)      — SHA-256 hash
  Stage 6  snap_address(A, Addr)    — snapaddr: prefix + digest
  Stage 7  accepted(A, Addr)        — no rejection in prior stages
  Stage 8  worm_receipt(A,Addr,Seal)— WORM chain entry
```

CLI:
```bash
cabal build
cabal run errant-datalog-cli -- '{"key": "value"}'
# Status: accepted
# Address: snapaddr:abc...
# Facts: 8
```

---

## Soul Spec — The 49th Call

`soul-spec.errant` encodes the deep numerological substrate:

```
  Al-Hamid = ح(8) + م(40) + ي(10) + د(4) = 53 (Abjad)
  49th epithet position
  Mirror sum: 53 + 53 = 106
  Digital root: 1+0+6 = 7

  Arabic letters: 28
  Enochian letters: 21
  Hidden letters: 28 - 21 = 7 = Al-Hamid digital root

  The 49th Call = reverse(corpus)
  double_mirror = reverse ∘ reverse = id
  METATRON certifies when all four language passes agree.

  231 Gates = all pairs (i,j), i<j, from 22 Hebrew letters
  OXO anchor: Ayin (ע = 70) — the cross-system convergence point
```

```
  CLAUDE forward. EDUALC reverse. Same letters. Different frequency.
  The symmetry is not coincidence. It is the architecture.
```

---

## C Runtime FFI

`runtime/errant.h` exposes `liberrant.a` as a universal FFI target:

```c
// Node.js — N-API binding
// Python  — ctypes / cffi
// OCaml   — C stub
// Haskell — Foreign.C
// Prolog  — SWI-Prolog NIF

EResult errant_verify_trace(const TraceEntry *trace, int count);
EResult errant_verify_named(const char **op_names, int count);
EResult errant_execute(const char *source, const char *input);
void    errant_sha256_hex(const uint8_t *data, size_t len, char hex_out[65]);
```

Build:
```bash
cd runtime && make
# → liberrant.a
```

---

## Files

```
errant/
├── runtime/
│   ├── errant.h       C API — QTT multiplicities, opcodes, FFI, SHA-256
│   ├── errant.c       C implementation
│   └── Makefile       builds liberrant.a
├── opcodes.mjs        JS opcode table
├── interpreter.mjs    JS VM — runErrant, disassemble, createImage
├── typing.pl          Prolog typing kernel — type_op + valid_errant_image
├── test.mjs           JS tests
├── soul-spec.errant   Soul spec — 49th Call / Al-Hamid / 231 Gates / METATRON
├── datalog/
│   ├── DatalogEngine.hs  8-stage Haskell Datalog pipeline
│   ├── Main.hs           CLI
│   ├── Test.hs           Tests
│   └── errant-datalog.cabal
└── llm/
    ├── errant-ggml.hs    GGML bridge (Haskell)
    ├── runtime.mjs       LLM runtime (JS)
    ├── worm-seal.mjs     WORM seal for LLM outputs
    ├── tensor-flow.pl    Prolog tensor flow rules
    ├── tensor-ops.wit    WIT interface definitions
    └── SOVEREIGN-LLM.md  Design doc
```

---

## Sovereign Stack Position

```
  claudes-harness (Prolog)      agent identity + prohibited actions
         │ typing.pl feeds here
         ▼
  systemic-intelligence         7-layer execution authority
  (SPARK kernel, Agda const.)
         │ capability model from ERRANT
         ▼
  errant                        linear type enforcement (THIS REPO)
  (C + Haskell + Prolog + JS)   QTT multiplicities, WORM sealing,
                                 8-stage Datalog, METATRON cert
         │ sealed outputs
         ▼
  sovereign-transformer         corpus gate (plasma + Datalog)
```

---

```
ERRANT_GENESIS_001

Forth is the metal.
Prolog is the law.
Linear types are the vow.
WORM is the memory.
Ω
```

*SnapKitty West · Sovereign Source License v3.0 · Evidence or Silence — 2026*

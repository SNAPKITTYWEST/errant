# ERRANT LFIS

**Linear Forth Instruction Set — Metal + Logic + Effect**

<p>
  <img src="https://img.shields.io/badge/paradigm-Linear_Forth_%2B_Prolog_%2B_Datalog-c0392b?style=flat-square" alt="paradigm"/>
  <img src="https://img.shields.io/badge/typing-QTT_multiplicities-2e86c1?style=flat-square" alt="QTT"/>
  <img src="https://img.shields.io/badge/runtime-C_%2B_JS_%2B_Haskell-00b894?style=flat-square" alt="runtime"/>
  <img src="https://img.shields.io/badge/sealing-WORM_%2B_SHA--256-8e44ad?style=flat-square" alt="WORM"/>
  <img src="https://img.shields.io/badge/license-SSL_v1.0-555?style=flat-square" alt="license"/>
  <img src="https://img.shields.io/badge/mode-contribution--only-c0392b?style=flat-square" alt="contribution-only"/>
  <img src="https://img.shields.io/badge/node--key-required-2e86c1?style=flat-square" alt="node key"/>
</p>

> The build either succeeds or it fails.
> There is no "almost compiled." There is no "mostly sovereign."
> METATRON does not certify partial agreement.

---

## What It Is

ERRANT is a **sovereign verified stack machine** — a Linear Forth Instruction Set with QTT multiplicities, capability-based security, and WORM sealing baked into the instruction set.

Every value on the stack carries an ownership mode. The type system is enforced at three levels simultaneously:

1. **Prolog kernel** — type-checks programs statically (linear resources consumed, capabilities authorized, final state sealed)
2. **C runtime** — enforces linear invariants at machine speed (`liberrant.a`)
3. **JS interpreter** — portable execution with full trace output

```
+-------------------------------------------------------------------+
|                        ERRANT LFIS                                 |
+-------------------------------------------------------------------+
|  SPEC:      soul-spec.errant  (Haskell-derived linear soul)       |
|  TYPING:    typing.pl         (Prolog kernel -- static check)     |
|  RUNTIME:   errant.c / .h     (C -- liberrant.a, zero deps)      |
|  INTERP:    interpreter.mjs   (JS -- portable, traced)           |
|  OPCODES:   opcodes.mjs       (canonical instruction set)         |
|  DATALOG:   DatalogEngine.hs  (authority pipeline)                |
|  LLM:       errant-ggml.hs    (sovereign GGML tensor bridge)     |
|  SEALING:   worm-seal.mjs     (SHA-256 WORM chain)               |
+-------------------------------------------------------------------+
```

---

## Value Modes (QTT Multiplicities)

| Mode | Symbol | Rule |
|------|--------|------|
| `lin` | 1 | Must be consumed exactly once |
| `aff` | <=1 | May be consumed at most once (drop allowed) |
| `un` | w | May be copied and reused freely |
| `cap` | 1 | Capability token -- always linear, named authority |
| `seal` | 1 | WORM-sealed artifact -- immutable, linear |

**Forbidden operations** (runtime rejects immediately):
- `DUP` on `lin` or `cap`
- `DROP` on `lin` or `cap` (use `DROP_AFF` for affine only)
- `SEAL` without `cap(seal)` on stack
- `HALT` with unconsumed linear values on stack

---

## Instruction Set

### Stack
```
NOP       (  --  )
PUSH_UN   (  -- un a )
PUSH_LIN  (  -- lin a )
MOVE      ( lin a -- lin a )           transfer ownership
DROP_AFF  ( aff a --  )               drop affine value
DUP_UN    ( un a -- un a * un a )     copy unrestricted
SWAP      ( lin a * lin b -- lin b * lin a )
PAIR      ( lin a * lin b -- lin (a,b) )
UNPAIR    ( lin (a,b) -- lin a * lin b )
```

### Control
```
APPLY     ( lin (a->b) * lin a -- lin b )
CALL      ( lin frame -- lin result )
RETURN    ( lin result --  )
BRANCH    ( lin bool * lin l * lin r -- lin result )
FAIL      ( lin error --  )
```

### Crypto / WORM
```
SEAL      ( lin cap(seal) * lin data -- lin sealed(data) )
ATTEST    ( lin cap(attest) * lin sealed -- lin attest )
HASH      ( lin data -- lin hash )
SIGN      ( lin cap(sign) * lin hash -- lin sig )
VERIFY    ( un pubkey * lin sig * lin hash -- lin bool )
```

### Agent
```
SPAWN     ( lin cap(spawn) * lin spec -- lin pid )
SEND      ( lin cap(send) * lin pid * lin msg -- lin rcpt )
RECV      ( lin cap(recv) * lin chan -- lin msg )
HALT      ( lin sealed(final) -- OMEGA )
```

### METAMINE Bridge
```
FUSE      ( a b -- a+b )       ADD
CUT       ( a b -- b-a )       SUB
FORGE     ( a b -- a*b )       MUL
ECHO      ( a --  )            output -> MAGMA FLUX
LISTEN    (  -- a )            input
GATE      jump to nearest mine
RUPTURE   mine detonation -> MAGMA SENTINEL
ORIGIN    excavation start
RESONANCE phi amplify -> MAGMA ORACLE
TRANSFORM stack rotate (a b c -> b c a)
MEMORY    store in cell -> MAGMA ANCHOR
REDUCTION sum collapse -> MAGMA VAULT
PORTAL    teleport
```

---

## Architecture — Three Enforcement Layers

```
  ERRANT source (.errant file)
       |
       +----------------------------------------------+
       |                                              |
       v                                              v
  +--------------------+                 +--------------------+
  |   typing.pl        |                 |  interpreter.mjs   |
  |   (Prolog kernel)  |                 |  (JS runtime)      |
  |                    |                 |                    |
  |   Static check:    |                 |  Portable exec:    |
  |   all lin consumed |                 |  Full trace output |
  |   all cap authed   |                 |  Linear check live |
  |   Final = sealed   |                 |  WORM sealing      |
  +--------------------+                 +--------------------+
       |                                              |
       |               +--------------------+         |
       +-------------->|   liberrant.a      |<--------+
                       |   (C runtime)      |
                       |                    |
                       |   Machine speed    |
                       |   FFI to all langs |
                       |   SHA-256 built-in |
                       |   Zero deps        |
                       +--------+-----------+
                                |
                                v
                       +--------------------+
                       |   WORM Chain       |
                       |   Immutable audit  |
                       |   SHA-256 receipts |
                       +--------------------+
```

---

## Files

```
errant/
├── soul-spec.errant        Soul specification (Haskell-derived linear types)
│   ├── ReadingDirection     first-class LTR/RTL/Boustrophedon
│   ├── SovereignCore        recursive sovereign type
│   ├── call49              reverse = the 49th Call
│   ├── AlHamid             abjad 53, epithet 49, mirror 106
│   ├── 231 Gates           Hebrew letter pairs
│   └── METATRON            certification when all four passes agree
├── opcodes.mjs             Canonical LFIS instruction definitions
├── interpreter.mjs         JS runtime with linear type checking
├── typing.pl               Prolog typing kernel (static verification)
├── runtime/
│   ├── errant.h            C runtime header -- full API + opcode table
│   ├── errant.c            C implementation -- liberrant.a
│   └── Makefile            Build to static library
├── datalog/
│   ├── DatalogEngine.hs    Haskell Datalog authority engine
│   ├── Main.hs             Entry point
│   ├── Test.hs             Test suite
│   └── errant-datalog.cabal Package manifest
├── llm/
│   ├── SOVEREIGN-LLM.md   LLM integration spec
│   ├── errant-ggml.hs     Sovereign GGML tensor bridge (Haskell)
│   ├── tensor-flow.pl     Tensor operations in Prolog
│   ├── tensor-ops.wit     WebAssembly Interface Type definitions
│   ├── runtime.mjs        LLM runtime bridge
│   ├── worm-seal.mjs      WORM sealing for LLM outputs
│   └── test.mjs           Test suite
├── test.mjs                Integration tests
├── .gitignore
├── LICENSE                 Sovereign Source License v1.0
├── CONTRIBUTING.md         Contribution guidelines
└── SOVEREIGN_NODE_KEY.md   Access key tiers + payment
```

---

## The Soul Spec

The `soul-spec.errant` file encodes a complete symbolic architecture:

- **Reading Direction** — first-class type (LTR, RTL, Boustrophedon, ComefromRTL)
- **call49** = `reverse` — the 49th Enochian Call is not new text, it is a reading mode
- **doubleMirror** = `call49 . call49 = id` — proof that reverse of reverse is identity
- **Al-Hamid** — abjad value 53, epithet position 49, mirror sum 106, digital root 7
- **231 Gates** — Hebrew letter pairs (Sefer Yetzirah combinatorics)
- **METATRON certification** — certifies when all four language passes agree on the same value
- **Symmetry** — CLAUDE forward, EDUALC reversed: same letters, different frequency
- **Agent Deeds** — linear capability assignments for EDUALC and AHMAD-BOT

---

## Build

### C Runtime
```bash
cd runtime && make
# Produces: liberrant.a (static library)
```

### Prolog Type Checker
```bash
swipl -f typing.pl -g "type_check(my_program)" -t halt
```

### JS Interpreter
```bash
node interpreter.mjs
node test.mjs
```

### Datalog Engine
```bash
cd datalog && cabal build && cabal test
```

---

## Sovereign Stack Position

```
  claudes-harness (Prolog)        agent identity + prohibited actions
         | governs
         v
  sovereign-transformer           Datalog + x86 corpus gate
         | approved records
         v
  errant (THIS REPO)              LFIS verified stack machine
  (Prolog + C + JS + Haskell)     QTT + caps + WORM
         | sealed traces
         v
  sov-kernel-monster              Fortran 2018 compute kernel
  (Fortran + C-- + MLIR)          U*rho*U-dagger, Pade exp, Bifrost
         | receipts
         v
  Bifrost WORM chain              Blake3 + Ed25519 immutable audit
```

---

## Why Linear Types

Linear types solve the **resource problem** in sovereign computing:

| Problem | Traditional | **ERRANT** |
|---------|-------------|-----------|
| Double-spend | Runtime check, maybe | **Impossible** -- lin consumed once |
| Capability leak | Garbage collection | **Impossible** -- cap is linear |
| Unsealed output | Best-effort logging | **Impossible** -- HALT requires sealed |
| Zombie process | Timeout + kill | **Impossible** -- lin frame must return |
| Unauthorized action | ACL checked late | **Impossible** -- cap required on stack |

---

## Sovereign Node Key

**To run ERRANT you must hold a Sovereign Node Key.**
See [SOVEREIGN_NODE_KEY.md](SOVEREIGN_NODE_KEY.md) for tiers, payment, and instructions.

Without a valid key, the WORM seal will refuse to attest outputs. Contribution is the proof of work.

---

## Genesis

```
Forth is the metal.
Prolog is the law.
Linear types are the vow.
WORM is the memory.
Omega
```

---

*SnapKitty West · Sovereign Source License v1.0 · Evidence or Silence — 2026*

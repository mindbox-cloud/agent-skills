# Maturity Stage Legend

> Reference for Exhausted Vitaly's contextual comments. Novice **does not compute** the maturity stage and does not determine the "actual stage" of a skill. This file is only for Exhausted Vitaly to contextualize findings relative to the user's declared goal.

---

## Stages

| Stage | Question | Value | Key Limitation |
|:---|:---|:---|:---|
| **1. Auto-generated** | Is this not garbage? | ~0: the model paraphrases what it already "knows" | Self-generated skills provide no improvement (SkillsBench) |
| **2. Personal skill** | Does the skill work for the author? | Captures one working path, reduces stochastic variance | Tied to the author's machine, permissions, data, and habits |
| **3. Team skill** | Can a colleague find, understand, and run it? | Portability, discoverability, stable navigation | No lifecycle artifacts, not embedded in an ecosystem |
| **4. Repository skill** | Does the skill live in an ecosystem and not rot? | Ownership, versioning, routing, lifecycle hygiene | Requires time investment; justified for skill collections |

---

## Practical Notes

- **Stage 2 — Personal skill.** Captures a working path and removes stochastic variance. Sufficient if the skill is only for you.
- **Stage 3 — Team skill.** A colleague can find, understand, and run it without verbal explanation. The main stage 2 danger is hidden coupling to the author's machine.
- **Stage 4 — Repository skill.** Owner, version, changelog, place in the ecosystem. The skill outlives its author. Changelogs and version headers matter here — they track modification history and make continuous support possible without depending on the original author's memory.
- **Important:** not every skill needs to be stage 4. A personal template is fine at stage 2.

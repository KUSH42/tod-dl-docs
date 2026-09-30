# TOD-DL documentation

[Repository](../README.md) / Documentation

![TOD-DL: resumable acquisition, durable state, verifiable custody][banner]

TOD-DL combines resumable acquisition, durable recovery, and signed custody
records. These guides explain how to use the software and how to evaluate
its engineering.

## Find your starting point

Choose a guide by the question you need to answer.

| Question | Guide |
| --- | --- |
| What problem does the project solve? | [Project brief][ref-1] |
| What engineering decisions can I review? | [Design and evidence][ref-2] |
| How do I run, resume, and verify an acquisition? | [Operator guide][ref-3] |
| Which component owns each responsibility? | [Architecture](ARCHITECTURE.md) |
| How do I work on the code? | [Development](DEVELOPMENT.md) |
| How do signed records establish a verifiable history? | [Audit rail][ref-4] |
| How do bytes move into a verified handover? | [Chain of custody][ref-5] |
| Which specification owns a behavior? | [Specification guide][ref-6] |

## Read at three depths

The documentation offers three levels without requiring a full spec read.

1. Read the [project brief](PORTFOLIO-OVERVIEW.md) for the problem, design
   choices, and validation evidence.
2. Follow the [architecture](ARCHITECTURE.md) or [custody walkthrough](CHAIN-OF-CUSTODY.md)
   for component boundaries and failure behavior.
4. Open the relevant [specification](SPECIFICATIONS.md), source file, and
   test when you need the exact contract.

## Understand the evidence

Guides explain the implementation. Specifications own behavior contracts.
Their `Status:` lines state implementation progress. Dated reports record
what a particular validation run checked.

A passing historical report does not establish that the current checkout
passes. A partially implemented specification can describe both available
behavior and planned extensions. [Open work](../specs/OPEN-WORK.md) names
remaining gaps; the linked spec defines the requirement.

## Next steps

Start with the [project brief](PORTFOLIO-OVERVIEW.md), or open the
[operator guide](OPERATOR-GUIDE.md) when you have an authorized case to run.

[banner]: assets/tod-dl-banner.png
[ref-1]: PORTFOLIO-OVERVIEW.md
[ref-2]: PORTFOLIO-OVERVIEW.md#engineering-decisions
[ref-3]: OPERATOR-GUIDE.md
[ref-4]: AUDIT-RAIL.md
[ref-5]: CHAIN-OF-CUSTODY.md
[ref-6]: SPECIFICATIONS.md

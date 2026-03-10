# h-network RFCs

Open specifications for AI agent safety, infrastructure automation, and multi-agent coordination — born from production experience, published for the community.

These RFCs document architectural patterns and safety frameworks developed during the building of [h-cli](https://github.com/h-network/h-cli), an open-source natural language infrastructure management platform. They are published here so others can implement, critique, extend, and build on them.

## RFCs

| RFC | Title | Status | Date |
|-----|-------|--------|------|
| [ASA-001](RFC-ASA-001.md) | The Asimov Safety Architecture for Autonomous AI Agents | Draft | March 2026 |

## What This Is

The AI agent space is moving fast. New frameworks ship daily. What's missing isn't more code — it's shared architectural knowledge about the hard problems: safety, trust boundaries, conflict resolution, and audit.

These RFCs are an attempt to formalize patterns that work in production. They use RFC 2119 language (MUST, SHOULD, MAY) not because they're IETF submissions, but because precision matters when you're specifying safety-critical behavior.

Every RFC published here has a reference implementation. Theory without code is speculation.

## Who This Is For

- Engineers building AI agents with tool access or shell execution
- Security teams evaluating autonomous agent deployments
- Framework authors looking for safety architecture patterns
- Anyone who's watched a single LLM talk itself out of its own safety rules

## How to Use These

**Implement.** Each RFC includes conformance requirements. Build against them.

**Challenge.** Open an issue if you find a gap, a contradiction, or a better approach. These are drafts, not dogma.

**Extend.** The architectures are designed for extensibility. Custom gates, custom layers, domain-specific rules — the specs tell you where the extension points are.

## Reference Implementations

| RFC | Implementation | Description |
|-----|---------------|-------------|
| ASA-001 | [h-cli](https://github.com/h-network/h-cli) | Natural language infrastructure management with the Asimov Safety Architecture |

## Contributing

If you've implemented an RFC and want your implementation listed, open a PR. If you want to propose a new RFC, open an issue with the problem statement first — the discussion matters more than the document.

## License

All RFCs in this repository are Copyright (c) 2026 h-network. Distribution is permitted in any form, provided attribution to the original author is preserved.

## Author

**Halil Ibrahim Baysal** — [h-network](https://h-network.nl) · [h-cli.ai](https://h-cli.ai) · [GitHub](https://github.com/h-network)

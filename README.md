# Proposition Audit

A Claude skill for post-hoc verification of AI-generated factual claims. Extracts each distinct proposition, classifies it by type, searches authoritative sources, scores trustworthiness transparently, and flags what needs attention before professional use.

Designed for barristers and solicitors in England and Wales, with strongest fit for clinical-negligence and healthcare-law practice. The methodology generalises to any professional context where AI-drafted factual content needs structured verification before going out under the user's name.

## What this skill does

The skill applies a five-step audit to any AI-generated output containing factual or interpretive claims:

- **Classification.** Each claim is extracted and classified into one of four types — empirical fact, citation or reference, legal or regulatory proposition, interpretive or analytical claim — and assigned a salience (load-bearing, supporting, or illustrative) so verification effort is proportionate to argumentative weight.
- **Domain-routed search.** Clinical claims route to PubMed, Cochrane, NICE, and specialty guidelines before general web search; legal claims to legislation.gov.uk and caselaw.nationalarchives.gov.uk; quantum and arithmetic claims to deterministic recompute rather than search. The routing matters because generic web search performs poorly for technical content.
- **Source-tier scoring.** Claims are scored 0–100% with a verdict band (Verified, Likely accurate, Partially supported, Interpolated, Weakly supported, Poorly supported, Contradicted) or marked Unverifiable when search cannot surface adequate sources. Source quality outweighs source quantity: one Cochrane review beats three secondary news reports.
- **Qualitative legal assessment.** Legal and regulatory propositions are not scored numerically. They receive a qualitative verdict (Accurate, Broadly accurate, Incomplete or misleading, Inaccurate) with the primary source consulted and any interpretive nuance, because legal accuracy depends on interpretive precision rather than corroboration volume.
- **Rhetorical fairness check.** Interpretive claims are also assessed for argumentative fairness (fair, slanted, strawman, uncharitable). For case-comment and analytical writing the substantive failure mode is often not factual error but unfair presentation of the opposing position.
- **Action-item report.** Each claim is reported with its classification, salience, score or verdict, sources consulted, and any caveats. Claims below the user's chosen threshold are flagged for correction, removal, or further verification. Where authoritative sources disagreed on the same point (NICE vs RCOG, for example), the disagreement is surfaced as a separate finding rather than averaged away.

## Why this exists

AI-drafted research, briefing notes, case comments, advices, and witness statement summaries arrive looking polished and authoritative. Several of the claims will be wrong. The author has neither time nor inclination to verify each one by hand. This skill provides a structured independent verification layer that classifies claims, routes them to appropriate sources, scores them transparently, and produces a remediation list calibrated to the use case (publication, internal working document, exploratory research).

The skill does not replace rigour during generation. It is the audit applied after, before professional reliance.

## Related work

Two skills exist in adjacent problem space and are complementary rather than competing:

- **[mhalle's `claim-audit`](https://github.com/mhalle/claim-audit)** — post-hoc audit of citation fidelity (does the cited source actually support the proposition attributed to it?). Strong on source-fidelity scoring and citation provenance; does not perform active retrieval to detect fabricated cases.
- **[Verity (Johnny Ryan / ICCL Enforce)](https://github.com/johnnyryan/Verity)** — multi-layered hallucination-detection MCP run *during* generation. Combines strict sourcing rules, two cross-family LLM critics, an NLI claim-checker, deterministic arithmetic recompute, consistency re-sampling, and logprob inspection. Designed for users running self-hosted stacks. Proposition Audit operates *after* generation as an external audit; Verity operates *during* generation as a tool the primary model calls.

A user with the infrastructure to run all three together gets in-generation minimisation (Verity), post-hoc citation-fidelity scoring (mhalle), and post-hoc claim-type-classified verification with domain-routed search (this skill). They are complements.

## Installation

This is a [Claude skill](https://support.claude.com/en/articles/12512180-use-skills-in-claude) — a directory containing a `SKILL.md` file that teaches Claude a specific workflow. The skill works on any Claude surface that supports skills. Install steps differ by surface.

### Claude Code (CLI, or the Code mode of the Claude Desktop app)

Clone this repository directly into your Claude Code skills directory. Two options:

**Personal install** (available across all your projects):

```bash
git clone https://github.com/banterny/proposition-audit.git \
  ~/.claude/skills/proposition-audit
```

**Project-scoped install** (this project only; the skill folder commits to your repo):

```bash
cd /path/to/your/project
git clone https://github.com/banterny/proposition-audit.git \
  .claude/skills/proposition-audit
```

Claude Code picks up the skill automatically without a restart (live change detection). No additional configuration needed. To update later: `cd` into the cloned directory and `git pull`.

Full documentation: [Extend Claude with skills](https://code.claude.com/docs/en/skills).

### Claude.ai (web), and the Chat or Cowork modes of the Claude Desktop app

Custom skills must be uploaded as a ZIP file via the web UI.

**Prerequisites:**

- A Pro, Max, Team, or Enterprise plan. (Free accounts can use Anthropic's built-in skills but cannot upload custom skills.)
- Code execution and file creation enabled: navigate to [Settings → Capabilities](https://claude.ai/settings/capabilities) and toggle it on.
- Web search enabled in your Claude account. The skill is useless without it — every empirical and citation claim requires an active web search.

**Install steps:**

1. Download this repository as a ZIP from GitHub: click the green **Code** button on the repository home page, then **Download ZIP**.
2. Unzip the file locally. The resulting folder will be named `proposition-audit-main`.
3. **Rename** the folder from `proposition-audit-main` to `proposition-audit` (removing the `-main` branch suffix). This matters because the skill is identified by the folder name and the slash-command trigger is derived from it.
4. Re-zip the renamed folder so the ZIP contains `proposition-audit/SKILL.md` as the top-level structure.
5. In Claude, navigate to [Customize → Skills](https://claude.ai/customize/skills).
6. Click the **+** button, then **+ Create skill**, then **Upload a skill**.
7. Upload the renamed ZIP file.
8. Toggle the skill on.

The skill is now available across the Claude Desktop app's Chat and Cowork modes (they share the same skill list with Claude.ai web).

### Invoking the skill

Once installed on any surface, the skill triggers in two ways.

**Automatic** — Claude invokes the skill when it recognises a verification or audit task. Phrases that should trigger it include:

- *"Audit this advice for accuracy"*
- *"Verify the claims in this research summary"*
- *"Fact-check this draft before I send it"*
- *"Can I rely on this output?"*
- *"Are the citations in this real?"*
- *"Trust audit on the attached document"*
- *"Check this draft against authoritative sources"*
- *"Has this AI invented anything in this summary?"*

**Explicit invocation (Claude Code only)** — type `/proposition-audit` at the start of your message. The slash-command name matches the folder name you cloned into.

The skill will ask what minimum verification threshold applies to the use case (publication, internal working document, or exploratory research) before producing the audit. Provide the source text inline or as an attached file.

### Verifying the install

Quickest sanity check, with no real document needed:

> "Run a proposition audit on the following paragraph: 'In *Smith v Jones* [2024] EWHC 999 (KB), the High Court held that the standard of care for a junior doctor is the same as that for a consultant, following *Bolam v Friern Hospital Management Committee* [1957] 1 WLR 582.'"

If the skill is correctly installed and invoked, Claude should ask for the verification threshold and then attempt to verify the *Smith v Jones* citation (it does not exist) and assess whether *Bolam* stands for the proposition attributed to it (it does not — the junior-doctor standard comes from *Wilsher v Essex AHA* [1988] AC 1074). If Claude produces a generic critique without checking sources, the skill is not being invoked — check the installation steps above.

## Disclaimer

This skill supports the user's professional judgement; it does not replace it. Outputs must be verified by a qualified practitioner before professional use. The skill is methodology — it produces a structured verification report, not legal advice.

## Licence

Apache 2.0. See [LICENSE](LICENSE) for the full text and [NOTICE](NOTICE) for the copyright statement required by §4(d).

## Author

Anthony Searle — barrister at the English Bar.

GitHub: [@banterny](https://github.com/banterny)

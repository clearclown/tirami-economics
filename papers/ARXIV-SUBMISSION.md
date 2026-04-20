# arXiv submission checklist — `compute-standard.arxiv.tar.gz`

> Ops guide for submitting the Tirami economic specification to arXiv.
> The user must execute this manually — submission requires a personal
> arXiv account and license acceptance that the maintainers cannot delegate.

## Prerequisites

1. **arXiv account** with submission privileges (free). If this is your
   first submission, you will need an endorsement from an existing arXiv
   author in the chosen category — see
   https://arxiv.org/help/endorsement.
2. **ORCID** linked to the arXiv account (recommended, not required).
3. **Build artifacts present.** From `tirami-economics/`:
   ```bash
   ls -la papers/build/compute-standard.arxiv.tar.gz
   ls -la papers/build/compute-standard.arxiv.pdf
   ```
   If missing, rebuild:
   ```bash
   cd papers && make clean && make arxiv
   ```

## Chosen category

**Primary:** `cs.DC` — Distributed, Parallel, and Cluster Computing.
**Cross-list (recommended):** `econ.GN` — General Economics,
`cs.CR` — Cryptography and Security.

Rationale: the paper straddles distributed systems (5-layer mesh, P2P
trade ledger), cryptographic primitives (Ed25519, zkML roadmap, slashing),
and economic theory (CU-native monetary system, Proof of Useful Work).
`cs.DC` is the strongest primary anchor; `econ.GN` catches the
monetary-theory audience; `cs.CR` catches the zkML / proof-of-inference
audience.

## Submission form — field by field

1. **Title:** `Tirami: Compute as the Unit of Account — An Economic Specification for Distributed LLM Inference`
2. **Authors:** use the real-name identity you want attached to the paper;
   arXiv enforces real-name policy.
3. **Abstract:** lift verbatim from the first section of
   `papers/compute-standard.md`. ~200–250 words.
4. **Comments:** `13 sections + 2 appendices, ~7,000 words. v0.1 working
   draft; companion implementation at github.com/clearclown/tirami.`
5. **Categories:** primary `cs.DC`, cross-list `econ.GN`, `cs.CR`.
6. **License:** `CC BY 4.0`. Matches the repository's content licence and
   lets readers redistribute / build upon.
7. **Upload:** `papers/build/compute-standard.arxiv.tar.gz`. arXiv's
   processor will unpack, run the included `compute-standard.arxiv.tex`
   through its own `xelatex` pipeline, and either accept or return errors.

## Test before submitting

```bash
# 1. Verify the tarball extracts cleanly
mkdir -p /tmp/arxiv-test && cd /tmp/arxiv-test
tar -xzf <repo>/papers/build/compute-standard.arxiv.tar.gz
ls  # expect compute-standard.arxiv.tex + any supporting .cls / .sty / image files

# 2. Local arXiv-style build (needs xelatex + the same fontspec stack the tarball assumes)
xelatex compute-standard.arxiv.tex && xelatex compute-standard.arxiv.tex
# expect: no fatal errors, a pdf at compute-standard.arxiv.pdf
```

If the local build succeeds, the arXiv build almost certainly succeeds.
If it fails, investigate the missing package / font before uploading —
arXiv's error output is much terser than a local toolchain's.

## Expected failure modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Package fontspec Error: The font "X" cannot be found` | Font not in arXiv's texlive | Swap to Latin Modern (`\setmainfont{Latin Modern Roman}` or remove `\setmainfont` entirely — the preamble in `header-arxiv.tex` already does this) |
| `Conversion errors (PDF not generated)` | Non-Unicode character or BOM in .tex | Re-encode source as UTF-8 without BOM: `file papers/build/compute-standard.arxiv.tex` should report `Unicode text, UTF-8` with no BOM |
| Endorsement required | First submission in `cs.DC` | Request endorsement from a published co-author or colleague with arXiv privileges |

## After submission

1. arXiv assigns an identifier within ~1 business day (announced overnight
   UTC). The pre-announcement URL is private to you.
2. Once announced, update the following to link the arXiv ID:
   - `tirami-economics/README.md` — add arXiv badge + link near the
     "論文 PDF" section.
   - `tirami-economics/docs/en/README.md` — same.
   - `tirami/README.md` — add an "arXiv preprint" line in the Docs
     section.
   - `tirami/docs/launch-phase19.md` — replace the GitHub-only references
     with the arXiv DOI.
3. File a v0.2 on arXiv (supersedes v0.1) if the theory chapters change
   materially before the paper is cited anywhere. arXiv versioning is free
   and preserves all prior versions.

---

## Related: Base Sepolia deploy

The on-chain deploy walkthrough lives with the contracts it describes:
see `tirami/docs/deploy-base-sepolia.md` (created alongside this file).

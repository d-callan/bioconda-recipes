# AGORA Bioconda Recipe

## Summary
This recipe packages AGORA (Algorithm for Gene Order Reconstruction in Ancestors), a Python-based tool for reconstructing ancestral genomes from comparative genomics data.

## Design Decisions

### Source & Versioning
- **GitHub commit archive** instead of PyPI/releases: AGORA has no official releases or git tags. The source is pinned to commit `f9dd78faab145bacb1808ed468b37c3cfb84fadd` (master, 2025-08-03), which corresponds to v3.1 as declared in the README and scripts.
- **Recommendation for upstream**: Consider creating a GitHub release tag for v3.1 (or v3.2 when ready) to enable cleaner versioning in bioconda and other package managers.

### Installation Strategy
- **`noarch: generic`**: Pure Python, no compiled code.
- **Preserve directory structure**: The entire source tree is copied to `$PREFIX/share/agora/` because:
  - AGORA is not a proper Python package (no `setup.py`)
  - Scripts use relative imports (`import utils.myAgoraWorkflow`)
  - The workflow engine discovers sub-scripts via `os.path.dirname(os.path.abspath(__file__))`, so the `src/` layout must remain intact
- **Wrapper scripts in `$PREFIX/bin/`**: All `src/*.py` scripts get bash wrappers that set `PYTHONPATH` to include `src/`, allowing direct invocation from the command line.

### Dependencies
- **Python ≥3.5**: Matches AGORA's stated compatibility (though the README warns about Python 3.8+ on macOS, which is not a concern for Linux-based bioconda).
- **psutil**: Only external dependency (from `requirements.txt`); everything else is Python stdlib.

### Testing
- **Lint check**: Passes bioconda-utils linting (including `run_exports` requirement).
- **Functional test**: Runs `ALL.extractGeneFamilies.py` on bundled example data (Species.nwk + GeneTreeForest.nhx.bz2), which:
  - Parses 11k gene trees
  - Extracts ancestral gene families for 4 ancestors + 5 extant species
  - Writes bz2-compressed output
  - Exercises core file I/O and phylogenetic tree logic in ~5 seconds

### Licensing
- **Dual GPL-3.0-or-later OR CeCILL-2.0**: Both are recognized open-source licenses. CeCILL-2.0 is a French law equivalent of GPL-3.0 (FSF-compatible), commonly used by French research institutions. Dual licensing allows choice based on jurisdiction. No redistribution restrictions for bioconda or Galaxy.

## Files
- `meta.yaml`: Recipe metadata, dependencies, test commands
- `build.sh`: Installation logic (copy + create wrappers)

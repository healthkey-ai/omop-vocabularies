# omop-vocabularies

The OMOP (Athena) vocabulary bundle and the LOINC release that
[promop](https://github.com/healthkey-ai/promop) loads at deploy time, published
as Git LFS objects so a deployment can fetch them over plain HTTPS without a
Google account, `gdown`, or Drive quota.

| File | What |
|---|---|
| `vocabulary_download_v5_{…}_<timestamp>.zip` | The Athena vocabulary export: `CONCEPT.csv`, `CONCEPT_RELATIONSHIP.csv`, … |
| `Loinc_<version>.zip` | The LOINC release, from which promop loads the six LOINC axes |
| `manifest.json` | What the two files are: name, SHA-256 and size, plus the LOINC version |

## How promop uses it

promop resolves a release by fetching `manifest.json` (one small request) and
compares the SHA-256s with what it has installed. Only when they differ does it
download the archives, from GitHub's LFS media endpoint:

```
https://raw.githubusercontent.com/healthkey-ai/omop-vocabularies/main/manifest.json
https://media.githubusercontent.com/media/healthkey-ai/omop-vocabularies/main/<file>
```

Set `ATHENA_VOCABULARY_LFS_URL=https://github.com/healthkey-ai/omop-vocabularies`
(append `/tree/<tag>` to pin a release). promop falls back to the Google Drive
folder in `ATHENA_VOCABULARY_GDRIVE_URL` when this repository cannot be reached.
See `docs/vocabularies.md` in promop.

## Publishing a new release

```bash
git lfs install
git clone https://github.com/healthkey-ai/omop-vocabularies.git && cd omop-vocabularies
git rm -q *.zip                      # one Athena bundle and one LOINC release at a time
cp /path/to/vocabulary_download_v5_*.zip /path/to/Loinc_*.zip .
# From a promop checkout: rewrite manifest.json with the new names and hashes.
DEBUG=True python manage.py shell -c "
from omop_core.services.vocabulary_source import build_manifest
build_manifest('/path/to/omop-vocabularies', released='YYYY-MM-DD')"
git add -A && git commit -m "Athena <build>, LOINC <version>"
git tag athena-YYYYMMDD-loinc-<version>
git push && git push --tags
```

Keep exactly one Athena ZIP and one LOINC ZIP in the tree: the manifest names
the files and promop's identity check keys on their SHA-256s, so a stale copy
beside a new one only costs LFS storage.

GitHub LFS limits: 2 GB per file; storage and bandwidth are metered per
organization. The Athena bundle is about 1 GB, so a deploy that has to download
it spends 1 GB of the bandwidth quota; a deploy whose vocabulary is current
spends nothing beyond the manifest.

## Licences

The archives carry their own terms, which this repository does not change:

- OHDSI Athena vocabularies: see the OHDSI [vocabulary licence
  notes](https://athena.ohdsi.org). Some member vocabularies (SNOMED CT, CPT4)
  have their own licensing conditions.
- LOINC: © Regenstrief Institute, Inc., distributed under the
  [LOINC and RELMA Terms of Use](https://loinc.org/license/).

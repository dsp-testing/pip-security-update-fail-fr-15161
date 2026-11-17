# Pipenv security-update failure reproduction

Reproduces the setup reported in [dependabot/dependabot-core#16896](https://github.com/dependabot/dependabot-core/issues/16896).

`Pipfile` and `Pipfile.lock` are copied byte-for-byte from
[`Sage-Bionetworks-IT/lambda-finops-floqast-sftp` at `e56a9993b283a22fe261e8ecde10f445b79e47c6`](https://github.com/Sage-Bionetworks-IT/lambda-finops-floqast-sftp/tree/e56a9993b283a22fe261e8ecde10f445b79e47c6).
The project requires Python 3.12. All four affected packages are transitive dependencies,
with different versions in the lockfile's two categories:

| Package | `default` | `develop` |
| --- | --- | --- |
| certifi | 2026.10.18 | 2026.4.22 |
| cryptography | 50.0.0 | 48.0.0 |
| idna | 3.15 | 3.13 |
| urllib3 | 2.8.0 | 2.6.3 |

## Reproduction

1. Keep the original manifests on the default branch; do not regenerate the lockfile.
2. Enable the dependency graph, Dependabot alerts, and Dependabot security updates.
3. Use `.github/dependabot.yml` to group all pip security updates while disabling routine version-update PRs.
4. Allow GitHub to ingest the lockfile and run the security update, then inspect the Dependabot run under Actions.

The reported failure is `RuntimeError: Expected Pipfile.lock to change!` while updating
the transitive dependencies, leaving security alerts open without a remediation PR.
The expected behavior is a grouped security-update PR fixing the vulnerable versions
in the `develop` category.

## Verified result

Reproduced on 2026-11-17 in
[Actions run 44083675290](https://github.com/dsp-testing/pip-security-update-fail-fr-15161/actions/runs/44083675290/job/134040450740)
using pipenv 2024.4.1 (Dependabot update job `1711998081`, fixture commit `e6f57fa`).

All four updates failed with `RuntimeError: Expected Pipfile.lock to change!`:

| Package | Attempted update |
| --- | --- |
| certifi | 2026.4.22 -> 2026.10.18 |
| cryptography | 48.0.0 -> 50.0.0 |
| idna | 3.13 -> 3.15 |
| urllib3 | 2.6.3 -> 2.8.0 |

The run ended with `Dependabot encountered '4' error(s) during execution`.
All [11 security alerts](https://github.com/dsp-testing/pip-security-update-fail-fr-15161/security/dependabot)
remained open, and no pull request was created.

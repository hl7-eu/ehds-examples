# EHDS FHIR Examples

This repository contains FHIR example content for EHDS-related implementation guide areas.

## Repository Structure

Top-level folders are organized by specification domain:

- `base`
- `eps`
- `hdr`
- `imaging`
- `lab`
- `mpd`

For pull requests, contributors should ideally create a dedicated subfolder for their examples inside the relevant top-level domain folder (for example, `lab/my-feature-examples/...`).

## Pull Request Validation

A GitHub Actions workflow (`FHIR Validation`) validates changes in pull requests.

Validation behavior:

1. Detects changed files in the PR.
2. Selects changed top-level spec folders from: `base`, `eps`, `hdr`, `imaging`, `lab`, `mpd`.
3. For each changed folder path in those domains, collects all `*.json` and `*.xml` files recursively from that folder.
4. Runs a precheck: each resource must have `meta.profile`, and at least one profile entry must start with `http://hl7.eu/fhir/`.
5. Runs the Java FHIR validator with:
   - `-allow-example-urls true`
   - `-html-output validation.html`
   - `-output validation.json`
   - `-show-message-ids`
   - `-extension any`
   - `-display-issues-are-warnings`
   - one `-resolution-context` per directory (including subdirectories) that contains JSON/XML content in the validated PR scope
   - required EHDS IG packages via `-ig`

Loaded IG packages:

- `hl7.fhir.eu.mpd#current`
- `hl7.fhir.eu.laboratory#2.0.0`
- `hl7.fhir.eu.imaging#1.0.0-ballot`
- `hl7.fhir.eu.eps#current`
- `hl7.fhir.eu.hdr#current`
- `hl7.fhir.eu.base#2.0.0`

### Validation Outputs in PRs

- A sticky PR comment is posted by a dedicated `workflow_run` workflow after `FHIR Validation` completes.
- The comment content is based on the rendered validation summary artifact and includes a download link to the validation artifact.
- Dependabot PRs are excluded from sticky comment posting.
- GitHub annotation output from the renderer.
- A downloadable validation artifact: `fhir-validation-html-report` (`validation.html`, `validation.json`).

## Failure Conditions

The `FHIR Validation` check fails when:
- at least one resource is missing `meta.profile`
- no `meta.profile` entry starts with `http://hl7.eu/fhir/` for at least one resource
- the Java validator reports errors (non-zero exit code)

## Local Validation Example

You can run a local validation manually with:

```bash
java -jar validator_cli.jar \
  -resolution-context lab/my-feature-examples \
  -resolution-context lab/my-feature-examples/subset-a \
  -ig hl7.fhir.eu.mpd#current \
  -ig hl7.fhir.eu.laboratory#2.0.0 \
  -ig hl7.fhir.eu.imaging#1.0.0-ballot \
  -ig hl7.fhir.eu.eps#current \
  -ig hl7.fhir.eu.hdr#current \
  -ig hl7.fhir.eu.base#2.0.0 \
  -allow-example-urls true \
  -html-output validation.html \
  -output validation.json \
  -show-message-ids \
  -extension any \
  -display-issues-are-warnings \
  lab/my-feature-examples/Observation-example-1.json \
  lab/my-feature-examples/subset-a/Observation-example-2.xml
```

Adjust `-resolution-context` and input files to your actual changed directories and files.

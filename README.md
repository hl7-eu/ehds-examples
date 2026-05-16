# EHDS FHIR Examples

This repository contains FHIR example content for EHDS-related implementation guide areas.

## Repository Structure

Top-level folders are organized by specification domain:

- `base`
- `eps`
- `imaging`
- `lab`
- `mpd`

For pull requests, contributors should ideally create a dedicated subfolder for their examples inside the relevant top-level domain folder (for example, `lab/my-feature-examples/...`).

## Pull Request Validation

A GitHub Actions workflow (`FHIR Validation`) validates changes in pull requests.

Validation behavior:

1. Detects changed files in the PR.
2. Selects changed top-level spec folders from: `base`, `eps`, `lab`, `mpd`.
3. For each changed folder path in those domains, collects all `*.json` and `*.xml` files recursively from that folder.
4. Runs the Java FHIR validator with:
   - `-allow-example-urls true`
   - `-html-output validation.html`
   - `-output validation.json`
   - `-show-message-ids`
   - `-extension any`
   - `-display-issues-are-warnings`
   - one `-resolution-context` per directory that contains changed/new content
   - required EHDS IG packages via `-ig`

Note: `imaging/**` is currently excluded from CI validation due to an API dependency issue.

Loaded IG packages:

- `hl7.fhir.eu.mpd#1.0.0`
- `hl7.fhir.eu.laboratory#2.0.0`
- `hl7.fhir.eu.imaging#current`
- `hl7.fhir.eu.eps#current`
- `hl7.fhir.eu.hdr#current`
- `hl7.fhir.eu.base#2.0.0`

### Validation Outputs in PRs

- A sticky PR comment with a Markdown summary rendered by `patrick-werner/validation-outcome-markdown-renderer`.
- The sticky PR comment includes a direct link to the uploaded HTML artifact.
- GitHub annotation output from the renderer.
- A downloadable HTML artifact: `fhir-validation-html-report` (`validation.html`).

## Failure Conditions

The `FHIR Validation` check fails when the Java validator reports errors (non-zero exit code).

## Local Validation Example

You can run a local validation manually with:

```bash
java -jar validator_cli.jar \
  -resolution-context lab/my-feature-examples \
  -resolution-context lab/my-feature-examples/subset-a \
  -ig hl7.fhir.eu.mpd#1.0.0 \
  -ig hl7.fhir.eu.laboratory#2.0.0 \
  -ig hl7.fhir.eu.imaging#current \
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

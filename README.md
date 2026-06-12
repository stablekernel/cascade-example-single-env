# cascade-example-single-env

A minimal, single-environment pipeline built with [cascade](https://github.com/stablekernel/cascade).

This example promotes one application straight to a single `prod` environment.
It shows the smallest useful cascade setup: one build callback, one deploy
callback, and the generated orchestration and release workflows.

## Layout

- `.github/manifest.yaml` - the pipeline definition (under the `ci` key).
- `.github/workflows/build-app.yaml` - reusable build callback. Emits an
  `artifact` output.
- `.github/workflows/deploy-app.yaml` - reusable deploy callback.
- `.github/workflows/orchestrate.yaml` - generated. Reacts to merges on `main`
  and opens a draft release.
- `.github/workflows/promote.yaml` - generated. Drives the release lifecycle
  (`create-draft`, `prerelease`, `release`) via manual dispatch.
- `.github/actions/manage-release/` - generated composite action used by the
  workflows.
- `.github/workflows/scenario-suite.yaml` - an end-to-end check that exercises
  the full release lifecycle and asserts on the resulting tags and releases.

## Regenerate

After editing the manifest, regenerate the workflows:

```bash
cascade generate-workflow --config .github/manifest.yaml --force
```

## Release lifecycle

1. Merge a change touching `src/**` to `main`. Orchestrate opens a draft
   release tagged `*-rc.0`.
2. Dispatch **Release** with `release_action: prerelease` to publish the
   release candidate for testing.
3. Dispatch **Release** with `release_action: release` to publish `vX.Y.Z`,
   mark it latest, and clean up the release-candidate tags.

The `scenario-suite` workflow runs the same three stages on a schedule and
asserts the expected tag and release shapes. It needs a `CASCADE_STATE_TOKEN`
repository secret.

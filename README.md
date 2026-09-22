# Atra workflow artifacts

This is the Atra artifact repository, separate from Atra product-code repositories. It stores one governed workflow run under `<Technology>/workflows/<feature-slug>/<WF-XXXX>/`.

Read [ARTIFACT-CONTRACT-v1.md](ARTIFACT-CONTRACT-v1.md) before adding or interpreting artifacts. The project index is [project.yaml](project.yaml). Technology names describe the work, not the agent or model assigned to it. `Cross-Stack` is for one concern spanning multiple technologies; it links to any narrower child workflows instead of duplicating authoritative decisions.

No workflow run is pre-created. A run begins only after an explicit Handoff request.

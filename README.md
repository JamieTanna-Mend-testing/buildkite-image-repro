# buildkite-image-repro

Reproduction repo for Renovate's support for detecting Docker images used
to run Buildkite agents, in the different places they can be defined in a
pipeline file.

Related context:

- [renovatebot/renovate#24017](https://github.com/renovatebot/renovate/issues/24017)
  - the original feature request
- [elastic/elasticsearch `.buildkite/pipelines/java-ea-check-new-build.yml`](https://github.com/elastic/elasticsearch/blob/62bdfa9c53cb3f5e8698ab16c3279df3b4f3a91c/.buildkite/pipelines/java-ea-check-new-build.yml#L4)
  and [elastic/thumbnails4j `.buildkite/deploy_snapshot_pipeline.yml`](https://github.com/elastic/thumbnails4j/blob/c39f7fc142c6b12de36f77d7f89c127138f174d0/.buildkite/deploy_snapshot_pipeline.yml#L5)
  - the two real-world examples cited in the issue, reproduced here as
    `pipeline.yml`/`pipelines/step-agents.yml`
- [Buildkite pipeline schema](https://raw.githubusercontent.com/buildkite/pipeline-schema/main/schema.json)
  - confirms `agents` can be a map or an array of `key=value` tags, and
    that pipeline-level/step-level `image` is a separate, documented field

No config is needed to enable the `buildkite` manager - it's enabled by
default (`renovate.json` here only sets `$schema`). It turns out the
`preview:buildkite` preset that used to document it as opt-in has been a
no-op since 2020, and has since been removed as part of this same work.

## Fixtures

- `.buildkite/pipeline.yml` - pipeline-root `agents.image`, as used by
  elastic/thumbnails4j, plus a plugin reference (`docker-compose#v4.16.0`)
  to check plugin extraction still works alongside image extraction.
- `.buildkite/pipelines/step-agents.yml` - step-level `agents.image`, as
  used by elastic/elasticsearch. This uses the same internal
  `docker.elastic.co` image referenced in the real pipeline, which isn't
  publicly resolvable - this fixture is about proving the dependency is
  *detected*, not that Renovate can find an update for it.
- `.buildkite/pipelines/agents-array.yml` - `agents` given as an array of
  `key=value` strings (rather than a map), at both pipeline root and step
  level.
- `.buildkite/pipelines/image-field.yml` - the official, documented
  pipeline-level/step-level `image` field (Kubernetes stack), distinct
  from the `agents.image` convention used elsewhere in this repo.
- `.buildkite/pipelines/group-step.yml` - a group step, to check images
  are found in nested `steps`, not just at the top level of a pipeline.

Docker images embedded in a plugin's own configuration (for example the
`docker` plugin's `image` key) are intentionally **not** covered here -
that's a separate, not-yet-implemented follow-up
(see `lib/modules/manager/github-actions/known-actions` for the pattern
it'll be modelled on).

## Testing locally

```sh
LOG_LEVEL=debug renovate --platform=local
```

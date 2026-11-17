# hawk-ecr-login

A GitHub Action that logs a job in to push task images to its repository's own protected task image repository on a [Hawk](https://github.com/METR/hawk) stack. The stack's operator opts the repository in first; [Protected task images](https://hawk.metr.org/infrastructure/task-images/#opting-a-repository-in) has the steps and a full publishing workflow.

```yaml
permissions:
  contents: read
  id-token: write
steps:
  - uses: METR/hawk-ecr-login@<commit> # <release>
    with:
      requester-role: <the stack's task_image_requester_role_arn output>
      broker: <the stack's task_image_broker_alias_arn output>
      audience: <the stack's task_image_github_audience output>
  - run: docker buildx bake --push # Tags ${DOCKER_IMAGE_REPO}:<task>-<version>.
```

Pin a release's commit: `git ls-remote https://github.com/METR/hawk-ecr-login refs/tags/<release>` prints it.

It needs `curl`, `jq` and AWS CLI v2. It sets `DOCKER_IMAGE_REPO` to the repository to tag images with, and points `DOCKER_CONFIG` at a config holding only this login, which lasts three hours: set up buildx after it, and log in anywhere else after each call. On a persistent self-hosted runner, don't call it from a `container:` job running as root.

Test: `uv run --isolated --no-project --with pytest --with pyyaml python -m pytest test_action.py`

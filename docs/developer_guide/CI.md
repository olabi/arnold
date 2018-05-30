# The CI

Historically, this project has been initiated using our own GitLab instance,
hence, we designed a GitLab-CI-based continuous integration (CI) workflow.
When we decided to open-source this project and move it to GitHub,
we lost our CI.

As it is a prerequisite for this project (for every project in fact),
As we usually use CircleCI, let's stick to it for Arnold.
We have improve and re-implement it. We have the advantage that CircleCI
and GitLab-CI philosophies are quite close.

# The Linters

## Dockerfile Linter
[Haskell Dockerfile Linter](https://github.com/hadolint/hadolint) help us to
build a [best practice](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
Docker image. It also use [ShellCheck](https://github.com/koalaman/shellcheck)
to lint the Bash code inside `RUN` instructions.

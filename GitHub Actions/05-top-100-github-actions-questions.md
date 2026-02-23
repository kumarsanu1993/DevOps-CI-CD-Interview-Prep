# 05-top-100-github-actions-questions.md: Staff-Level Interview Prep

This document contains 100 staff-level GitHub Actions questions, complete with concise answers and official reference links.

## Architecture & Runners Setup (1-20)

**1. How do you implement scalable ephemeral runners in Kubernetes?**
**Answer**: By deploying Actions Runner Controller (ARC) using a `RunnerDeployment` custom resource. ARC auto-scales pods based on webhook events.
**Reference**: [Actions Runner Controller](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/about-actions-runner-controller)

**2. What is the fundamental difference between GitHub-hosted and self-hosted runners regarding state?**
**Answer**: GitHub-hosted runners provide a clean, isolated VM for every job. Self-hosted runners maintain state between jobs unless explicitly cleaned or configured as ephemeral.
**Reference**: [About self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)

**3. How do you share data between jobs in the same workflow?**
**Answer**: Using `actions/upload-artifact` in the upstream job and `actions/download-artifact` in the downstream job.
**Reference**: [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

**4. How can you trigger a workflow from an external system securely?**
**Answer**: Use the `repository_dispatch` event, calling the GitHub API with a Personal Access Token (PAT) and a custom `client_payload`.
**Reference**: [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch)

**5. How do you manage a monolithic repository where only the modified microservice gets built?**
**Answer**: Using a combination of `paths:` or `paths-ignore:` at the workflow trigger level, or utilizing tools like `dorny/paths-filter` to dynamically check paths in a setup job.
**Reference**: [Workflow syntax - on.<push|pull_request>.paths](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore)

**6. Explain the mechanism for reusing identical CI steps across hundreds of repositories.**
**Answer**: Create centralized Reusable Workflows (using `on: workflow_call`) in an organizational `.github` repository.
**Reference**: [Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

**7. What is a Composite Action and when should you use it over a Reusable Workflow?**
**Answer**: A Composite Action bundles multiple *steps* into a single step, defined in `action.yml`. Use it for step-level modularity (e.g., standardizing a setup sequence) rather than full job/workflow modularity.
**Reference**: [Creating a composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

**8. How do you pass secrets to a Reusable Workflow?**
**Answer**: You must explicitly pass them using the `secrets:` keyword when calling the workflow, or use `secrets: inherit` to pass all calling workflow secrets automatically.
**Reference**: [Reusing workflows - passing secrets](https://docs.github.com/en/actions/using-workflows/reusing-workflows#passing-inputs-and-secrets-to-a-reusable-workflow)

**9. How does `concurrency` work to prevent overlapping deployments?**
**Answer**: By defining `concurrency: group_name` at the workflow or job level. Adding `cancel-in-progress: true` aborts historical runs in favor of the newest.
**Reference**: [Using concurrency](https://docs.github.com/en/actions/using-jobs/using-concurrency)

**10. How do you implement a dynamic matrix strategy where the array is generated at runtime?**
**Answer**: Use an upstream job to evaluate the required targets, set them as a JSON array string in `$GITHUB_OUTPUT`, and use `${{ fromJson(needs.job1.outputs.matrix) }}` in the downstream matrix job.
**Reference**: [Using a matrix for your jobs](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)

**11. What is the execution limit per job on GitHub-hosted runners?**
**Answer**: 6 hours. After this, the job is forcibly terminated.
**Reference**: [Usage limits, billing, and administration](https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration#usage-limits)

**12. How do you ensure a workflow runs only when a PR is merged, not just closed?**
**Answer**: Trigger on `types: [closed]` under `pull_request`, and add a job-level `if: github.event.pull_request.merged == true` condition.
**Reference**: [Events that trigger workflows - pull_request](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request)

**13. How can you deploy an application across multiple AWS regions simultaneously using Actions?**
**Answer**: Define a `matrix` strategy with region codes and deploy them in parallel jobs, referencing `${{ matrix.region }}`.
**Reference**: [Using a matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)

**14. What is the scope limitation of the automatically generated `GITHUB_TOKEN`?**
**Answer**: It is scoped only to the repository where the workflow is running. It cannot modify files in other repositories.
**Reference**: [Automatic token authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)

**15. How do you fetch a private GitHub repository module inside a workflow?**
**Answer**: Create a Personal Access Token (PAT) or GitHub App token, add it as a secret, and pass it to the `actions/checkout` step using `with: token: ${{ secrets.PAT }}`.
**Reference**: [actions/checkout](https://github.com/actions/checkout)

**16. What dictates the default shell on Windows GitHub runners?**
**Answer**: PowerShell (`pwsh`).
**Reference**: [Workflow syntax - jobs.<job_id>.steps[*].shell](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsshell)

**17. How do you debug a workflow without continually committing to main?**
**Answer**: Enable step debug logging by setting repository secret `ACTIONS_STEP_DEBUG` to `true`, and use tools like `act` to run the workflow locally.
**Reference**: [Enabling debug logging](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging)

**18. What is the `workflow_dispatch` event and what unique feature does it offer?**
**Answer**: It allows manual triggering via UI/API and uniquely allows defining `inputs` (parameters) that the user must fill out before the run.
**Reference**: [Providing inputs for manually triggered workflows](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatchinputs)

**19. Can a workflow trigger another workflow directly?**
**Answer**: Yes, through `workflow_run`, `workflow_call` (reusable workflows), or by firing a `repository_dispatch` API call. However, actions performed with the default `GITHUB_TOKEN` will not trigger new workflows to prevent recursive loops.
**Reference**: [Triggering a workflow from a workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow)

**20. What is an Environment in GitHub Actions?**
**Answer**: A logical deployment target (e.g., Staging, Prod) that allows you to configure protection rules (like requiring manual approvals) and environment-specific secrets.
**Reference**: [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)

## Security & IAM (21-40)

**21. Why is OIDC highly recommended for AWS deployments over IAM User keys?**
**Answer**: OIDC uses short-lived, dynamically generated tokens, eliminating the risk of committing static AWS keys and dealing with credential rotation.
**Reference**: [Configuring OpenID Connect in Amazon Web Services](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)

**22. How do you implement OIDC with AWS in Actions?**
**Answer**: Set permissions to `id-token: write` and `contents: read`, then use `aws-actions/configure-aws-credentials` specifying the generic Role ARN to assume.
**Reference**: [OIDC AWS Configuration](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)

**23. What are the major risks of running workflows on `pull_request_target`?**
**Answer**: `pull_request_target` runs the workflow in the context of the base branch, meaning it has access to secrets. If an attacker submits a PR altering the build script, it could execute malicious code and steal secrets.
**Reference**: [Keeping your GitHub Actions and workflows secure](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/)

**24. How do you mitigate Supply Chain attacks on Third-Party Actions?**
**Answer**: Pin the action to a specific immutable full commit SHA instead of a mutable tag (e.g., `@v2`), and use Dependabot to update the SHA safely.
**Reference**: [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions#using-third-party-actions)

**25. How do you generate SLSA Provenance for an artifact in Actions?**
**Answer**: Using the SLSA framework's generic generator GitHub Actions or native language integrations (e.g., `slsa-github-generator`) to output strictly signed attestations attached to the GitHub Release.
**Reference**: [SLSA GitHub Generator](https://github.com/slsa-framework/slsa-github-generator)

**26. How do you force a workflow to be executed on every new repository created in an organization?**
**Answer**: Define standard organization-level "Required Workflows" enforced via rulesets or GitHub Enterprise policies.
**Reference**: [About required workflows](https://docs.github.com/en/actions/using-workflows/required-workflows)

**27. What is `CODEOWNERS` and how does it relate to Actions?**
**Answer**: A file specifying the owners of specific paths in a repo. Using Actions environments, you can enforce that Environment Deployments specifically require approvals from users in `CODEOWNERS`.
**Reference**: [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)

**28. How do you securely authenticate Dependabot PRs knowing they do not have access to repository secrets?**
**Answer**: Store specific secrets strictly for Dependabot under Repo Settings -> Secrets -> Dependabot.
**Reference**: [Configuring access to private registries for Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/configuring-access-to-private-registries-for-dependabot)

**29. What is least-privilege token configuration?**
**Answer**: Forcing the `GITHUB_TOKEN` to explicitly deny privileges unless specified via the `permissions:` block at the job/workflow level (e.g., `contents: read`).
**Reference**: [Assigning permissions to jobs](https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs)

**30. How can you scan Docker images built in CI for vulnerabilities before pushing?**
**Answer**: Execute tools like Trivy or Anchore via their GitHub Actions wrapper steps, integrating output to the GitHub Advanced Security dashboard.
**Reference**: [Trivy vulnerability scanner](https://github.com/aquasecurity/trivy-action)

(Due to markdown sizing, questions 31-100 continue in the same staff-level architecture...)

**31. How do you securely pass a Database connection string dynamically to Terraform via Actions?**
**Answer**: Store it in HashiCorp Vault or AWS Secrets Manager, use OIDC to authenticate, fetch the secret at runtime into memory as an environment variable, and never output it to a log.

**32. What dictates if a secret is masked in GitHub Action logs?**
**Answer**: GitHub automatically masks values matching the exact string of registered secrets. However, if a secret is formatted (e.g., base64 encoded by the script), it will leak in plaintext.

**33. How do you detect leaked secrets automatically?**
**Answer**: Enable "Secret Scanning" via GitHub Advanced Security which checks both commits and PRs before they merge.

**34. Can self-hosted runners be used securely on open-source public repositories?**
**Answer**: Highly discouraged unless the runner is explicitly ephemeral, strictly isolated via VMs (no Docker access), and on an air-gapped network, because any PR can execute arbitrary code.

**35. What is the difference between organizational and repository secrets?**
**Answer**: Org secrets can be shared across multiple/all repositories, reducing the maintenance overhead. Repo secrets are siloed natively to one repository.

**36. Explain the `environment` protection wait timer.**
**Answer**: You can configure an environment to have a "Wait timer" natively pausing the deployment for a set interval (e.g., 10 minutes) after pushing, allowing manual aborts.

**37. How can you limit SSH access to an ephemeral Jenkins/Action Runner pod?**
**Answer**: Run the pod with a tightly controlled SecurityContext (non-root), deny SSH ingress, and rely purely on pipeline logs forwarded to Datadog/Splunk.

**38. How is the GitHub Actions Token structurally different from a PAT?**
**Answer**: The automatically generated GITHUB_TOKEN is essentially a temporary GitHub App installation token expiring at job completion (max 24 hrs), whereas a PAT persists until manually revoked or expired.

**39. Can you sign commits executed by the GitHub Actions bot?**
**Answer**: Yes, by importing a GPG key into a secret and using an action (`crazy-max/ghaction-import-gpg`) to sign the commits as the bot.

**40. Are workflow logs encrypted at rest?**
**Answer**: Yes, GitHub encrypts all workflow logs and artifacts at rest.

## Optimization & Caching (41-60)

**41. Why might `actions/cache` fail to speed up a Docker build?**
**Answer**: The GitHub cache acts on the runner's filesystem. Standard Docker builds inside the runner lack visibility to it. For Docker, utilize `--cache-from` combined with a Docker registry cache backend.

**42. How does the cache Key work?**
**Answer**: An exact hit on the key retrieves the cache. A "restore-keys" array performs partial matching (e.g., OS and prefix) to grab the most recent base cache if the primary key misses.

**43. Is GitHub Cache shared between different branches?**
**Answer**: A branch can access its own caches, the base branch's caches (e.g., `main`), but NOT sibling branches' caches.

**44. How do you clear a corrupted GitHub Actions cache?**
**Answer**: Use the GitHub CLI (`gh extension install actions/gh-actions-cache`) or use the GitHub Desktop/Web UI to manually locate and delete the specific cache key.

**45. What is the maximum size allowed for caches per repository?**
**Answer**: 10 GB limit per repository (across all caches). Exceeding this evicts the oldest caches.

**46. How do you cache Maven/Gradle dependencies effectively?**
**Answer**: Cache the `~/.m2` or `~/.gradle` folder using a hash of the `pom.xml` or `build.gradle` file as the key.

**47. When using ARC in Kubernetes, how do you handle Docker-in-Docker layer caching?**
**Answer**: Utilize a sidecar proxy with a mounted Persistent Volume Claim (PVC) or leverage external remote cache backends (AWS S3) via Buildkit.

**48. Why is using `fetch-depth: 0` in `actions/checkout` important for tools like SonarQube?**
**Answer**: Default checkout only fetches the single latest commit. SonarQube requires full git history to accurately determine code blame and new code coverage.

**49. How do you profile slow workflow jobs?**
**Answer**: Monitor the timestamps of the workflow step execution in the log. Complex profiling requires dumping the metrics into an exporter (e.g., OpenTelemetry action).

**50. What is "fail-fast" in matrix strategies?**
**Answer**: A setting (`fail-fast: true` by default) where if ANY matrix job fails, all other executing matrix jobs are instantly cancelled to save compute.

**51. How does GitHub allocate bandwidth for downloading packages in runners?**
**Answer**: GitHub-hosted runners are on Azure cloud, offering massive bandwidth. However, external API rate limiting (like Docker Hub's pull limits) often causes throttling.

**52. How do you prevent Docker Hub rate limits in GitHub Actions?**
**Answer**: Authenticate via `docker login` step with Docker Hub credentials, or proxy pulls through GHCR / AWS ECR.

**53. How do you parallelize a massive Cypress E2E test suite?**
**Answer**: Use a matrix build indexing test suites dynamically, or natively utilize Cypress Dashboard parallelization tokens injected as secrets.

**54. What tool is best to check YAML syntax of complex workflows before pushing?**
**Answer**: The `actionlint` CLI tool can catch variable interpolation errors, syntax issues, and permission mismatches locally.

**55. How do you prevent an artifact upload if the previous step failed?**
**Answer**: Provide the condition `if: always()` or `if: success()` appropriately on the artifact step.

**56. What is the impact of running self-hosted runners on spot/preemptible instances?**
**Answer**: Huge cost savings, but jobs will randomly fail when the Spot VM is reclaimed. Ensure jobs are idempotent and pipelines auto-retry.

**57. How to specify an immutable action environment regarding Node versions?**
**Answer**: Use `.nvmrc` or `.node-version` file populated with a strict version and configure `actions/setup-node` to read it via the `node-version-file` input.

**58. How do you execute steps based on changed files accurately?**
**Answer**: Use the `tj-actions/changed-files` action to output which files changed, routing logic through conditional steps.

**59. What happens if a job outputs data that is larger than standard env constraints?**
**Answer**: Limit output data formats; use `actions/upload-artifact` instead of `env` variable buffers to transfer massive strings.

**60. What's the fastest way to deploy static sites using Actions?**
**Answer**: Generating the site to an artifact and directly deploying using GitHub Pages native deployment actions (`actions/deploy-pages`).

## Enterprise Governance & APIs (61-80)

**61. How does Enterprise GitHub manage Action runner groups?**
**Answer**: Enterprise allows grouping self-hosted runners so specific teams can access high-compute GPU runners while others default to basic ones.

**62. How can you automate the creation of repository secrets?**
**Answer**: Utilize the GitHub REST API or official Terraform provider `github_actions_secret` blocks.

**63. What API does `gh` CLI use to trigger jobs?**
**Answer**: The GitHub REST API v3 specifically targeting the `dispatches` endpoint.

**64. How do you aggregate code coverage across a matrix of 10 Node shards?**
**Answer**: Upload the separate lcov files as artifacts, wait for them to finish using `needs`, and run a dependent job to group them and merge using tools like `nyc merge`.

**65. How do you enforce mandatory branch protections blocking merges until a specific action passes?**
**Answer**: In Branch Protections, assign a "Required Status Check" pointing explicitly to the Job name in the Action.

**66. What logic is required to post a PR comment automatically after Terraform Plan?**
**Answer**: Output the terraform plan to a file, read the output in a subsequent step, and use `actions/github-script` (which injects the `github` octokit client) to create an issue comment dynamically.

**67. Is there a charge for GitHub Action minutes on public repos?**
**Answer**: No, public repositories on GitHub have unlimited free action minutes.

**68. How do you inject a random UUID into an action step?**
**Answer**: Execute `uuid=$(uuidgen)` in a bash step and append it to `$GITHUB_ENV`.

**69. How does Github Advanced Security (GHAS) hook into Actions?**
**Answer**: Using the `github/codeql-action/init`, building the project, and then `github/codeql-action/analyze` to generate SARIF data uploaded straight to the UI.

**70. Can you use Actions to build ARM64 images?**
**Answer**: Yes, but since standard GitHub-hosted Linux runners are AMD64, you must use QEMU (`docker/setup-qemu-action`) and Buildx to emulate the ARM instruction set, which is slower.

**71. How do you configure a runner to NOT automatically update its agent?**
**Answer**: Supply `--disableupdate` to the self-hosted runner configuration script.

**72. Can jobs run across different repositories sequentially?**
**Answer**: Theoretically via API calls, but standard `needs:` syntax only works for jobs within the *same* workflow execution.

**73. What are GitHub Services (Containers)?**
**Answer**: Defining auxiliary containers (like PostgreSQL or Redis) natively in the job configuration under `services:`. They run alongside the job for integration testing.

**74. How does `services` networking work?**
**Answer**: The runner creates a user-defined bridge network; steps inside the runner can access the service using the service's defined label (e.g., `postgres:5432`).

**75. How do you execute Python scripts utilizing native GitHub actions API?**
**Answer**: Write the python script, install PyGithub via pip in a setup step, and pass the `GITHUB_TOKEN` to it as an environment variable to interact with PRs.

**76. Using Terraform in Actions, how do you handle state locking without concurrency?**
**Answer**: Utilize backend state locking (like DynamoDB for AWS), but wrap the pipeline in a `concurrency:` group to prevent multiple workflow executions crashing into each other trying to lock it.

**77. Explain "Job Summary" functionality.**
**Answer**: A markdown report appended centrally to the Workflow Run UI dynamically by appending markdown text to `$GITHUB_STEP_SUMMARY`.

**78. How do you force GitHub to rebuild an image, ignoring Docker cache completely?**
**Answer**: Pass the `--no-cache` parameter during the docker build command executed inside the pipeline shell step.

**79. What distinguishes GitHub Actions billing limits from other platforms?**
**Answer**: macOS runners consume minutes at 10x the rate of Linux runners; Windows consumes at 2x.

**80. Can you deploy directly from GitHub Actions to an isolated VPC?**
**Answer**: Only if you use an AWS VPN/Direct Connect attached Self-Hosted Runner positioned INSIDE the network, or set up AWS Systems Manager (SSM) agents.

## Troubleshooting & Debugging (81-100)

**81. Why does my workflow not trigger on pushing a tag?**
**Answer**: Your `on: push:` event must explicitly define `tags:` (e.g., `tags: ['v*.*.*']`).

**82. Why is my multiline string secret getting corrupted?**
**Answer**: Using standard `echo` on multiline environment variables truncates newlines. Use native Environment File writing with EOF markers explicitly.

**83. How do you fix "Permission denied" when executing a bash shell script inside your repo?**
**Answer**: Run `git update-index --chmod=+x script.sh` to give it execution rights in Git, or run `chmod +x` in an upstream action step.

**84. Job says "No space left on device", how do you solve it without self-hosting?**
**Answer**: Delete unnecessary tools from the GitHub-hosted Ubuntu runner using a community action prior to running your heavy step, saving up to 20GB of disk space.

**85. What causes a `detached HEAD` state during Actions?**
**Answer**: `actions/checkout` defaults to checking out the exact commit SHA that triggered the workflow, causing a detached head state. To push changes back, you must explicitly check out a branch.

**86. Why can't Dependabot workflows trigger subsequent workflows?**
**Answer**: To prevent recursive execution and abuse, workflows triggered directly by Dependabot's token execution cannot trigger other events like `workflow_run`.

**87. How do you securely capture Action runtime errors without breaking the build?**
**Answer**: Use `continue-on-error: true` at the step level, assign an `id` to the step, and query its `steps.<id>.outcome` in downsteam logic.

**88. "Invalid workflow file" syntax errors are common; what's the strict requirement for `if:` conditionals regarding context encapsulation?**
**Answer**: In `if:` statements, wrapping variables in `${{ }}` is optional and sometimes syntactically disallowed if placed inside functions incorrectly.

**89. Why is a specific job not appearing in the dependency graph (`needs`)?**
**Answer**: Typo in the `needs:` array string, or a conditional `if:` rule evaluated to false upstream, skipping it entirely (which causes dependent jobs to also skip).

**90. How do you trace an API rate limit breach back to GitHub Actions?**
**Answer**: Query GitHub Enterprise Audit logs looking for the specific Personal Access Token acting on behalf of the automated service account.

**91. Why does `${GITHUB_REF}` show a strange ref instead of branch name during Pull Requests?**
**Answer**: In a PR event, `GITHUB_REF` is `refs/pull/:prNumber/merge`. Using `GITHUB_HEAD_REF` is required to get the actual branch name.

**92. How do you solve 'RPC failed; curl 56 GnuTLS recv error' when checking out massive monorepos?**
**Answer**: Tweak Git HTTP post buffer size: run `git config --global http.postBuffer 524288000` before checkout, or use shallow clones.

**93. Can a step access `env` variables defined in a previous *job*?**
**Answer**: No. Jobs run on separate machines. Data must be explicitly passed via job `outputs` or `artifacts`.

**94. What is the limit of job outputs?**
**Answer**: A job can define up to 100 outputs, and the maximum length of an output value is 1 MB.

**95. Workflow dispatch is missing from the Actions UI dropdown; why?**
**Answer**: A workflow with `workflow_dispatch` MUST be committed and merged to the default branch (usually `main`) at least once before the UI button appears.

**96. Why did a matrix job run 256 times and crash the organization pipeline queue?**
**Answer**: Providing two matrix arrays of size 16 generates a Cartesian product (16 x 16 = 256 jobs). Be explicit to prevent explosion or use `include:` specifically.

**97. How does using nested reusable workflows affect dependency visibility?**
**Answer**: The caller workflow sees the reusable workflow as a single job block block. It cannot easily query the intermediate outputs of nested jobs inside the reusable file.

**98. Why do environment protections not trigger for specific jobs?**
**Answer**: The `environment:` property must be explicitly bound to the *Job* that performs the deployment. If it's missing, no rules act upon it.

**99. Is it possible to trigger an Action upon deleting a branch?**
**Answer**: Yes, use the `on: delete` webhook event condition. 

**100. How do you gracefully tear down infrastructure when a workflow is cancelled manually?**
**Answer**: Use the `if: cancelled()` conditional to attach a cleanup step that runs regardless if the user pressed the red "Cancel Workflow" button.

*(Note: References to official documentation: [GitHub Actions Docs](https://docs.github.com/en/actions) / [GitHub Community](https://github.com/orgs/community/discussions))*

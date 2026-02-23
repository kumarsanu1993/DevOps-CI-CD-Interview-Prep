# 05-top-100-gitlab-ci-cd-questions.md: Expert-Level Interview Prep

This document contains 100 expert-level GitLab CI/CD questions, complete with concise answers and official reference links.

## Pipeline Architecture & Modularity (1-20)

**1. What is the fundamental difference between GitLab CI and Jenkins regarding configuration?**
**Answer**: GitLab CI is natively Declarative (YAML) and deeply integrated into the Git repository ecosystem without requiring a separate master server to configure GUI jobs.
**Reference**: [GitLab CI/CD Concepts](https://docs.gitlab.com/ee/ci/concepts/)

**2. Explain the execution phases of `.gitlab-ci.yml`.**
**Answer**: GitLab first parses the YAML to validate syntax, then "compiles" the pipeline creating the DAG/Sequential stages, provisions the required runners dynamically, and executes the isolated jobs.
**Reference**: [Pipeline Architecture](https://docs.gitlab.com/ee/ci/pipelines/)

**3. How do you implement a DRY pipeline architecture across hundreds of microservices?**
**Answer**: By heavily utilizing the `include:` keyword, pointing to a centralized `ci-templates` repository containing `.yml` fragments, and injecting variables.
**Reference**: [include keyword](https://docs.gitlab.com/ee/ci/yaml/includes.html)

**4. What is the difference between `extends` and YAML Anchors?**
**Answer**: YAML anchors (`&`) and aliases (`*`) are standard YAML features and only work within the *same* file. `extends` is a GitLab-specific feature that works across merged dictionaries, even those pulled in via `include`.
**Reference**: [extends keyword](https://docs.gitlab.com/ee/ci/yaml/#extends)

**5. How do you avoid running backend tests when only frontend code changes?**
**Answer**: Use the `rules:changes` block, specifying paths (e.g., `src/frontend/**/*`), which dynamically evaluates the Git diff before adding the job to the pipeline.
**Reference**: [rules:changes](https://docs.gitlab.com/ee/ci/yaml/#ruleschanges)

**6. Explain how Parent-Child pipelines improve performance on monorepos.**
**Answer**: Instead of one monolithic pipeline compiling hundreds of jobs and hitting pipeline size limits, the parent `.gitlab-ci.yml` evaluates what changed and dynamically triggers smaller, isolated child pipelines (`trigger: include:`) only for affected modules.
**Reference**: [Parent-child pipelines](https://docs.gitlab.com/ee/ci/pipelines/parent_child_pipelines.html)

**7. Can a child pipeline trigger its own child pipeline?**
**Answer**: Yes, GitLab supports nested pipelines up to two levels of depth (Parent -> Child -> Grandchild).
**Reference**: [Nested child pipelines](https://docs.gitlab.com/ee/ci/pipelines/parent_child_pipelines.html#nested-child-pipelines)

**8. What is a Multi-Project Pipeline and how does it differ from a Child Pipeline?**
**Answer**: Multi-project pipelines trigger a workflow in an *entirely different repository* (`trigger: project: "my-group/my-infra"`), whereas child pipelines exist within the same repository codebase.
**Reference**: [Multi-project pipelines](https://docs.gitlab.com/ee/ci/pipelines/multi_project_pipelines.html)

**9. How do you generate a GitLab CI configuration dynamically at runtime?**
**Answer**: Use a job in an upstream stage to run a script (e.g., Python/Bash) that outputs a formatted `.yml` file as an artifact. A downstream job then parses that artifact using `trigger: include: artifact: generated.yml`.
**Reference**: [Dynamic child pipelines](https://docs.gitlab.com/ee/ci/pipelines/downstream_pipelines.html#dynamic-child-pipelines)

**10. What defines the order of Job execution within a Stage?**
**Answer**: Jobs within the identical stage always run in parallel, assuming there are enough idle runners. They do not execute sequentially unless forced via the `needs:` DAG keyword.
**Reference**: [Stages](https://docs.gitlab.com/ee/ci/yaml/#stages)

**11. What is a DAG (Directed Acyclic Graph) in GitLab CI?**
**Answer**: Using the `needs:` array to explicitly map job dependencies across stages, overriding the traditional "wait for the entire previous stage to finish" behavior, severely reducing overall pipeline execution time.
**Reference**: [Directed Acyclic Graph](https://docs.gitlab.com/ee/ci/directed_acyclic_graph/)

**12. How do you pass variables dynamically to a downstream Multi-Project pipeline?**
**Answer**: Store the variables in a `dotenv` file, upload it as a `reports: dotenv` artifact, and declare it under the `variables:` mapping of the `trigger:` block.
**Reference**: [Pass variables to a downstream pipeline](https://docs.gitlab.com/ee/ci/pipelines/downstream_pipelines.html#pass-variables-to-a-downstream-pipeline)

**13. What is Auto DevOps?**
**Answer**: A pre-configured collection of GitLab CI/CD templates that automatically detects, builds, tests, deploys, and monitors your applications leveraging Docker and Kubernetes organically.
**Reference**: [Auto DevOps](https://docs.gitlab.com/ee/topics/autodevops/)

**14. What are the major drawbacks of Auto DevOps?**
**Answer**: It is highly opinionated. If an enterprise deviates from the strict Heroku-style buildpacks, standard chart deployments, or requires complex pre-routing logic, modifying the massive underlying Auto DevOps scripts is extremely fragile.
**Reference**: [Customizing Auto DevOps](https://docs.gitlab.com/ee/topics/autodevops/customize.html)

**15. How do you manage deployment to specific Environments with approval gates?**
**Answer**: Define an `environment: name: production` variable within the job, and couple it with `rules: - when: manual`. Additionally, go to the GitLab UI -> Settings -> Environments to configure "Protected Environments" requiring explicit approvals from specific users.
**Reference**: [Protected environments](https://docs.gitlab.com/ee/ci/environments/protected_environments.html)

**16. What is the difference between `script` and `before_script`?**
**Answer**: `before_script` is typically defined globally or inherited via templates to set up the environment (like logging into registries). `script` contains the actual execution commands for the job itself. If `before_script` fails, the job fails immediately.
**Reference**: [before_script](https://docs.gitlab.com/ee/ci/yaml/#before_script)

**17. What happens if a job uses `allow_failure: true`?**
**Answer**: If the job exits with a non-zero code, it displays a yellow warning icon, but the pipeline continues traversing down through subsequent stages.
**Reference**: [allow_failure](https://docs.gitlab.com/ee/ci/yaml/#allow_failure)

**18. How do you configure a job to run automatically ONLY if a previous job failed?**
**Answer**: Use the `rules: - when: on_failure` block combined with a `needs:` pointing to the previous job, allowing native cleanup/rollback actions.
**Reference**: [rules:when](https://docs.gitlab.com/ee/ci/yaml/#ruleswhen)

**19. How do you execute jobs strictly on a schedule?**
**Answer**: Configure a Pipeline Schedule in the GitLab UI, and inside the `.gitlab-ci.yml`, use `rules: - if: '$CI_PIPELINE_SOURCE == "schedule"'` to explicitly target those runs.
**Reference**: [Pipeline schedules](https://docs.gitlab.com/ee/ci/pipelines/schedules.html)

**20. Explain the `interruptible` keyword.**
**Answer**: If `interruptible: true` is set, and a developer pushes a new commit while this job is currently running on the old commit, GitLab will aggressively cancel the old running job to save runner minutes.
**Reference**: [interruptible](https://docs.gitlab.com/ee/ci/yaml/#interruptible)

## Runners & Infrastructure Scaling (21-40)

**21. Architecturally, how does the GitLab Runner differ from Jenkins Agent?**
**Answer**: GitLab Runners constantly poll the GitLab API (outbound HTTPS) asking for work. The GitLab server does not push jobs to the runners.
**Reference**: [GitLab Runner architecture](https://docs.gitlab.com/runner/#runner-execution-flow)

**22. How do you isolate specific high-memory runners for specific teams?**
**Answer**: Assign unique tags to the runners during registration (e.g., `high-mem`, `gpu`). In the `.gitlab-ci.yml` job, add the `tags: - high-mem` array to force execution on those nodes.
**Reference**: [Use tags to control which jobs a runner can run](https://docs.gitlab.com/ee/ci/runners/configure_runners.html#use-tags-to-control-which-jobs-a-runner-can-run)

**23. What is the Docker executor capability vs Shell executor?**
**Answer**: The Docker executor spins up an ephemeral, clean container matching every job's `image:` tag. The Shell executor executes the script directly on the Runner host's raw operating system, potentially polluting it.
**Reference**: [Executors](https://docs.gitlab.com/runner/executors/)

**24. How do you implement Runner Autoscaling on Kubernetes?**
**Answer**: Using the Kubernetes executor. A core Runner Manager pod lives in the cluster. For every matched job in the queue, it speaks to the K8s API to spin up a dynamic Pod strictly for that job's lifetime.
**Reference**: [Kubernetes executor](https://docs.gitlab.com/runner/executors/kubernetes.html)

**25. How do you securely execute Docker (`docker build`) inside a Kubernetes Runner?**
**Answer**: Running Docker-in-Docker (dind) requires privileged pods, which breaks K8s security. The Expert-level solution relies on rootless daemon builders like Google's **Kaniko** or **Buildah**, which map filesystem outputs to container registries without requiring Docker socket access.
**Reference**: [Use Kaniko to build Docker images](https://docs.gitlab.com/ee/ci/docker/using_kaniko.html)

**26. How do you manage concurrent job limits per Runner?**
**Answer**: In the runner's `config.toml` file, there is a global `concurrent = X` setting dictating the maximum number of simultaneous jobs the whole manager handles, and an `[runners] limit = Y` setting dictating maximum concurrency per registered instance.
**Reference**: [Advanced configuration](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)

**27. What are the security risks of Shared Runners?**
**Answer**: In massive public groups, malicious actors can obfuscate cryptomining scripts inside pipelines burning down free computing tier minutes, forcing Gitlab to enforce strict quota limitations.
**Reference**: [Security considerations for shared runners](https://docs.gitlab.com/runner/security/)

**28. How does `clone` vs `fetch` caching work on runner machines?**
**Answer**: Configured via `GIT_STRATEGY`. `clone` aggressively wipes the whole directory and re-downloads the entire repo every single job (slow, safe). `fetch` re-uses the existing local `.git` directory and just pulls recent changes (fast, potentially risky if earlier scripts poisoned the tree).
**Reference**: [Git strategy](https://docs.gitlab.com/ee/ci/runners/configure_runners.html#git-strategy)

**29. How do you pass huge Docker Images between stages efficiently?**
**Answer**: You don't pass them via artifacts. Instead, build and push them to the GitLab Container Registry (using `${CI_REGISTRY_IMAGE}`) in the first stage, and then immediately `docker pull` them in the downstream stage.
**Reference**: [GitLab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/)

**30. Why might a runner appear "Stuck" constantly even if it sits Idle?**
**Answer**: Often due to tag mismatches. A job specifies a tag (`deploy`), but no runner registered to the project possesses that explicit tag AND the runner is configured to `run_untagged: false`.

(Continuing structurally...)

**31. Explain Runner 'Pre-clone' scripts.**
**Answer**: A hook inside `config.toml` that executes a standard bash command blindly before GitLab brings down the source code. Extremely useful for injecting global proxy certs deeply into the VM.

**32. What is the function of the GitLab Runner `cache_dir` parameter?**
**Answer**: Defining explicitly where on the Host system the massive zip archives for CI Caching are stored, allowing them to be offloaded to cheap spinning-disk volumes.

**33. How does the runner determine its authentication payload token?**
**Answer**: When registered, it generates a persistent runner token stored in the `config.toml`. It is utilized strictly to poll and attach workloads.

**34. Can one Runner agent register to multiple completely separate GitLab instances?**
**Answer**: Yes, the `config.toml` accepts an array mapping pointing to completely different URLs and tokens concurrently.

**35. What is the impact of placing the `.gitlab-ci.yml` file elsewhere?**
**Answer**: By default, it must reside exactly in the root. However, under Settings -> CI/CD -> General Pipelines, you can override the target manifest path to a nested folder (e.g., `.app/ci/main.yml`).

**36. How do you integrate AWS Secrets directly into a GitLab Job organically?**
**Answer**: By configuring a JWT/OIDC identity provider mapping between GitLab and AWS STS. The job organically parses AWS credentials without defining static environment API keys.

**37. Explain the function of `${CI_JOB_TOKEN}`.**
**Answer**: An ephemeral, automatic variable. It is heavily utilized interacting natively back with GitLab's own API (downloading internal artifacts, tagging releases) expiring instantly upon job termination.

**38. What is the consequence of outputting secrets directly to stdout?**
**Answer**: By default, GitLab implicitly attempts to mask any defined project variables matching exact variable strings out of the log.

**39. Can you dynamically select the runner based on changed variables?**
**Answer**: Yes, passing dynamic variables directly to the `tags:` array.

**40. Are runner cache files shared securely between separate projects?**
**Answer**: No. Strict isolation is enforced. Caches are distinctly siloed matching project IDs natively on disk.

## Optimization, Artifacts, and Caching (41-60)

**41. How does the `cache: key:` string dictate optimizations?**
**Answer**: Utilizing `$CI_COMMIT_REF_SLUG` enables a completely distinct cache purely for specific branches avoiding cache poisoning across parallel development tracks.

**42. Artifacts vs. Cache—what fundamentally survives execution failures?**
**Answer**: Cache is aggressively uploaded even on arbitrary job failures (assuming `when: always` on cache blocks). Artifacts strictly abort uploading if the job exits non-zero unless explicitly bypassed via `artifacts:when`.

**43. Is it ideal to cache `node_modules` globally?**
**Answer**: No. Cache strictly utilizing the hashing algorithm (`cache: key: files: - package-lock.json`) so it implicitly invalidates only when dependencies mutate.

**44. How handles uploading 5GB compiled binaries across stages natively?**
**Answer**: It usually fails. GitLab's default artifact max payload size is strictly minimal (e.g., 100MB-1GB) managed by administrators. Leverage S3 storage offloading for massive payloads utilizing raw AWS CLI.

**45. What does the `expire_in:` keyword solve natively?**
**Answer**: Aggressively garbage collects intermediate stage artifacts saving petabytes of enterprise disk space across heavy monorepos.

**46. Can you download artifacts from identical projects executing on parallel branches?**
**Answer**: Yes, utilizing the explicit internal API or utilizing the `needs: project:` keyword.

**47. Explain the consequences of setting `GIT_DEPTH: 1`.**
**Answer**: Executes a shallow clone pulling only the final commit. Monumentally speeds up massive repo pulls but breaks tools requiring historic commit inspection (e.g., standard SonarQube algorithms).

**48. Why does my Docker executor job take 5 minutes pulling the exact identical image concurrently?**
**Answer**: Ensure the runner's `config.toml` defines `pull_policy = "if-not-present"` explicitly mitigating redundant registry fetching.

**49. How do you skip Pipeline execution globally via commit message?**
**Answer**: Adding `[ci skip]` or `[skip ci]` strictly in the Git Commit message natively aborts pipeline instantiation.

**50. What triggers a "YAML Invalid" immediately post push?**
**Answer**: Syntax faults executing unquoted strings, misaligned colons, or violating internal keyword schemas parsed instantly upon ingestion.

**51. How does GitLab manage parallel matrix strategy job outputs?**
**Answer**: Each matrix branch operates identically in isolation. It prevents dynamic array output combining, requiring explicit down-stream data parsing jobs processing discrete outputs organically.

**52. Is it possible to define identical keys appending to global arrays utilizing `extends:`?**
**Answer**: Yes natively via array merging semantics.

**53. How do you implement robust timeout thresholds?**
**Answer**: Utilizing `timeout:` directly underneath a specific job bounding infinite testing freezes.

**54. What constitutes a Release in GitLab CI?**
**Answer**: Appending `.gitlab-ci.yml` `release:` configurations automatically appending executable payloads securely to the GitLab internal Releases page linked permanently to tag deployments.

**55. How do you monitor explicit Runner Metric data (CPU/Memory) locally?**
**Answer**: Enabling the embedded Prometheus listener directly inside the `config.toml`.

**56. How do you handle secrets effectively using HashiCorp Vault?**
**Answer**: Utilizing `secrets:` block internally resolving Vault endpoints utilizing the `$CI_JOB_JWT` identity tokens completely omitting UI UI configurations.

**57. What defines "Environment Variables Precedence" hierarchy natively?**
**Answer**: Trigger API > UI Variables > YAML `variables` block.

**58. How to securely expose UI variables explicitly strictly limiting exposure to development branches?**
**Answer**: Checking "Protected Variable" checkmarks forces mapping restrictions identical to protected branch Git logic.

**59. What prevents multiple pipelines crushing a staging deployment simultaneously?**
**Answer**: Enabling `resource_group:` organically bounds concurrency preventing parallel jobs acquiring concurrent deployment locks simultaneously.

**60. Can a job retry automatically strictly on specific API HTTP timeouts?**
**Answer**: Yes. Defining `retry: max: 2, when: runner_system_failure` narrowly focuses retry architectures mitigating random network timeouts bypassing actual unit-test codebase failures.

## Security, Compliance & OIDC (61-80)

**61. What is the fundamental architecture difference between GitLab JWT and standard Tokens?**
**Answer**: JWTs are strictly ephemeral and cryptographically signed affirming job metadata contexts uniquely accepted by foreign OIDC providers (Vault/AWS/GCP).

**62. How do you force mandatory security scanning executing globally without developer permission?**
**Answer**: Establishing Security/Compliance Pipelines structurally at Group/Instance level executing arbitrary YAML configs executing preceding `.gitlab-ci.yml` inclusions natively.

**63. Describe SAST (Static Application Security Testing) natively encapsulated within GitLab.**
**Answer**: Implementing the standard `template: Security/SAST.gitlab-ci.yml` injects language-specific parsing tools scanning for inherent CVEs organically projecting results internally against MR interfaces.

**64. Contrast DAST (Dynamic) scanning structurally internally vs SAST.**
**Answer**: SAST parses underlying raw codebase offline. DAST requires explicit deployments (staging environment) violently launching external HTTP penetration testing against active application targets.

**65. How do you verify dependency structures scanning directly for known outdated malicious software implementations natively?**
**Answer**: Enabling `Dependency-Scanning.gitlab-ci.yml` mapping specific package-lock files comparing globally against CVE databases.

**66. Explain Container Scanning inside parallel architectures.**
**Answer**: Post Docker-build payloads securely scanned executing standard Trivy/Clair engines natively analyzing OS packages embedded internally identifying distinct vulnerabilities seamlessly.

**67. What is Secret Detection?**
**Answer**: Executing automated hooks mapping RegEx logic detecting accidental API keys, AWS credentials natively pushed explicitly against specific branches.

**68. Describe the risks associated explicitly checking out codebase against public Shared runners.**
**Answer**: Data extraction. Explicitly compiling source code offline capturing explicit IP. Highly suggest explicitly scoping Runners utilizing Private tags specifically against exact IP data.

**69. How do you handle explicit Docker credential integrations natively without plaintext UI entries?**
**Answer**: Establishing DOCKER_AUTH_CONFIG formatted JSON structures natively injecting login definitions intrinsically mitigating hardcoded credential leakages globally.

**70. What dictates MR Approval thresholds mapping directly to pipeline outputs?**
**Answer**: Integrating GitLab Premium mappings restricting merging unless explicit Vulnerability outputs explicitly register zero Critical flaws globally affecting explicit targets.

**71. How do you limit runner exploitation preventing explicit Root access globally?**
**Answer**: Executing strict `privileged = false` constraints natively bound across `config.toml` ensuring container separation structures.

**72. Describe strategies logging all pipeline interactions natively integrating against Enterprise auditing environments.**
**Answer**: Configuring implicit GitLab Audit hooks mapping structured web-traffic directly indexing towards Splunk parsing pipeline execution matrices implicitly.

**73. What is generic Token abuse regarding `$CI_JOB_TOKEN` scopes?**
**Answer**: Bounding project scopes. If the token holds administrative mapping it actively modifies surrounding API configurations. Mitigate limiting implicit permission configurations intrinsically.

**74. How to protect variables aggressively isolating against forked repositories executed explicitly internally?**
**Answer**: MR's explicitly executing against forks implicitly drop protected variable variables intrinsically preventing arbitrary data execution paths inherently protecting environment configs.

**75. Difference between Masking vs Protective variable configurations structurally?**
**Answer**: Mask targets Log outputs obfuscating strings. Protective targets Branch mappings restricting underlying job access globally.

**76. How do you map specific SSH configurations cloning private repositories executing natively against Shell runners?**
**Answer**: Standardizing `ssh-agent` architectures embedded evaluating `$SSH_PRIVATE_KEY` globally evaluating specific known-hosts directly executing Git protocols dynamically.

**77. Explain GitLab Fuzz Testing structurally mapped inside CI ecosystems.**
**Answer**: Actively throwing infinite random payloads implicitly evaluating buffer overflows capturing explicit system crashes indexing specifically back.

**78. What inherently prevents explicitly tracking artifacts beyond limited retention cycles globally?**
**Answer**: Data decay scripts implicitly bound against default `expire_in` structures preventing unlimited payload sizes natively globally.

**79. How do you execute compliance environments verifying SLSA outputs implicitly mapping origins natively against artifacts?**
**Answer**: Generating SLSA attestations organically executing provenance scripts directly indexing artifact signatures explicitly proving secure origins intrinsically evaluating payloads.

**80. Can you lock `.gitlab-ci.yml` editing bypassing explicit developer access entirely?**
**Answer**: Removing specific commit access mappings structurally restricting file modification utilizing Code Owners natively targeting explicit Admin approvals.

## Troubleshooting & Debugging Edge Cases (81-100)

**81. Why do dynamically generated Child pipelines instantly fail stating valid YAML faults natively executed identically manually?**
**Answer**: Variable collision limits explicitly restricting implicit inheritance configurations requiring deep validations natively analyzing specific paths globally.

**82. What causes runner 'Image Pull' errors natively occurring dynamically despite executing perfect identical pipeline runs?**
**Answer**: Internal Registry rate limits throttling implicit unauthenticated payload structures globally executing concurrent builds.

**83. How do you diagnose explicitly stalling pipelines natively executing parallel environments hanging globally indefinitely?**
**Answer**: Uncaught standard input requests natively pausing background daemons demanding UI evaluations indefinitely halting script environments globally. (e.g., hanging `apt-get` without `-y`).

**84. Why does the UI implicitly drop Job Logs halfway executing compiling native components globally?**
**Answer**: Payload truncation thresholds. GitLab structurally abandons capturing explicit standard out data executing exceeding global data thresholds preventing internal database exhaustion natively.

**85. How do you execute local debugging mapping specifically validating underlying GitLab configurations locally?**
**Answer**: Executing `.gitlab-runner exec` natively analyzing implicit Docker definitions parsing identical architectures parsing identical files implicitly mimicking explicit cloud operations.

**86. What generates explicit `Missing Pipeline` executions natively despite executing explicit push architectures accurately?**
**Answer**: Exceeding global quota scopes structurally pausing execution contexts mapping administrative delays explicitly suspending executions natively.

**87. How handles resolving explicit `needs:` dependencies mapping specifically against explicit conditional jobs skipped implicitly globally?**
**Answer**: Utilizing `optional: true` explicitly mitigating cascading failures parsing explicit job graphs isolating dependencies parsing dynamic evaluations intrinsically.

**88. Why do global caches corrupt implicitly executed parallel environments natively deploying identical configurations?**
**Answer**: Concurrent execution mapping similar disk locations violently violating read/write matrices explicitly. Mapping Cache explicitly utilizing Job IDs mitigates concurrent overwrites dynamically.

**89. What defines explicit pipeline execution executing identically against `main` mapped completely uniquely against explicit tag deployments?**
**Answer**: `$CI_COMMIT_TAG` evaluations explicitly branching underlying `rules` architectures inherently swapping execution scopes globally manipulating payloads explicitly parsing underlying structures.

**90. How do you resolve internal Docker socket exhaustion intrinsically mapping executing explicit dind parallel matrices concurrently?**
**Answer**: Separating dind daemon structures globally explicit execution limits structurally prioritizing parallel volumes implicitly executing globally mitigating explicit container exhaustion paths.
**91. How do you bypass the 5MB `.gitlab-ci.yml` maximum payload YAML size limit organically in hyper-scale monorepos?**
**Answer**: Utilize `trigger: include: artifact` dynamically orchestrating downstream pipelines via generated subsets, or offload configuration schemas utilizing CI lint validation via separate lightweight trigger components.

**92. Why does a downstream Child Pipeline arbitrarily show as passed even if its internal jobs catastrophically failed?**
**Answer**: You must explicitly configure `strategy: depend` on the `trigger:` block; otherwise, the parent pipeline considers the trigger "successful" simply because it successfully *started* the child pipeline.

**93. Explain how underlying host Kernel mismatches cause silent Docker-in-Docker failures on self-hosted runners.**
**Answer**: If the `dind` container expects specific overlayFS structures or cgroups (v1 vs v2) that the underlying bare-metal runner's kernel lacks, Docker daemon startup fails silently or panics mounting volumes natively. 

**94. How do you prevent GitLab from aggressively rate-limiting a deployment script hitting the internal API thousands of times?**
**Answer**: Cache API payloads locally within the job runner using `curl` buffers, implement exponential backoff logic on the API calls, or structure the payload into bulk batched GraphQL queries.

**95. What dictates that a Runner completely ignores a `git push --force` destroying upstream histories?**
**Answer**: If a pipeline execution context relies upon previous commit SHAs mapping strict artifacts natively, a force push orphans the old pipelines, structurally causing internal 404s fetching historically retained payloads.

**96. How do you solve pipeline failures occurring strictly because `.gitlab-ci.yml` includes an external remote file that 502 temporarily timeouts?**
**Answer**: Architect resilience by mirroring required remote templates internally within the same GitLab instance preventing dependency on external HTTPS routes failing unpredictably.

**97. What causes a `needs:` block to violently fault even if the upstream job succeeded natively?**
**Answer**: If the upstream job executes only under specific `rules` that evaluating false causes it to be skipped, the downstream `needs` breaks entirely unless you append `optional: true`.

**98. How do you diagnose a runner execution failing explicitly pointing to "No space left on device" when `df -h` shows 500GB free?**
**Answer**: Inode exhaustion. Building excessive tiny files (e.g., massive Javascript node_modules) natively corrupts execution structures even if raw block storage space exists inherently.

**99. Why does `$CI_PIPELINE_ID` collision structurally corrupt external deployment tracking systems parsing webhooks?**
**Answer**: Pipeline IDs are instance-unique natively, but transferring deployments across entirely isolated GitLab servers requires mapping against universally unique commit SHAs rather than integer IDs.

**100. How do you orchestrate a true zero-downtime runner infrastructure upgrade locally without abruptly killing active CI Jobs?**
**Answer**: First, explicitly send a `SIGQUIT` signal to the `gitlab-runner` process initiating a Graceful Shutdown, causing it to stop accepting new workloads but allowing existing jobs perfectly complete beforehand natively.

*(Note: References to official documentation: [GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/))*

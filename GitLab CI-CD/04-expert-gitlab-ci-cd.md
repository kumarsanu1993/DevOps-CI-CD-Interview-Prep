# 04-expert-gitlab-ci-cd.md: Enterprise Operations & Security

## 👑 Expert Corner: Governance, Runner Autoscaling, and Compliance
At the Expert level, you are managing GitLab across the enterprise, ensuring compliance, and optimizing runner infrastructure.

---

## 1. Enterprise Runner Infrastructure

### Q1: How do you scale GitLab Runners efficiently?
- **Answer**: Utilizing Docker Machine (legacy) or the Kubernetes Executor.
- **Kubernetes Executor Pattern**:
  1. The Runner Manager lives in the K8s cluster.
  2. When a job is triggered, the Manager dynamically spins up a Pod to execute the job.
  3. You configure node autoscaling (e.g., Karpenter or Cluster Autoscaler) so the cluster grows when the CI queue spikes and shrinks back down when idle.

### Q2: What are some security considerations for maintaining your own runners?
- **Privileged Mode Security Risk**: If a `docker` executor is running in `privileged: true` mode (often required for Docker-in-Docker `dind`), the user/job has root access to the host machine.
- **Mitigation**: Use rootless Docker/Podman, or use Kaniko/Buildah for building Docker images inside CI without requiring privileged mode.

## 2. Security and Compliance

### Q3: What is a Compliance Pipeline?
- **Answer**: A feature in GitLab Ultimate where group administrators define a pipeline configuration in a centralized repository that *forcibly runs* for all projects in a specific framework/group.
- **Use case**: Enforcing security scanning (SAST/DAST) or mandatory audit logs for every deployment, regardless of what the individual repo's `.gitlab-ci.yml` says.

### Q4: How does GitLab integrate with Vault/OIDC?
- **Answer**: Instead of permanently storing production credentials in GitLab CI variables (which can be risky), GitLab provides a JSON Web Token (JWT) linked to the pipeline.
- **Workflow**: The job uses the JWT to authenticate to HashiCorp Vault (or AWS via OIDC), and Vault issues a short-lived token to perform the deployment. The credentials expire automatically.

## 3. GitLab Registry & Auto DevOps

### Q5: What is Auto DevOps?
- **Answer**: GitLab's built-in collection of CI/CD templates that automatically detect the language, build the application, test it, run code quality scans, build a Docker image, and deploy it to Kubernetes—often with zero configuration needed.
- **Pros**: Incredible out-of-the-box experience to hit the ground running.
- **Cons**: Too opinionated for complex, heavily decoupled architectures.

### Q6: How does GitLab Container / Dependency Registry integrate?
- **Answer**: Highly integrated via `${CI_REGISTRY}` variables. You can push artifacts directly to the GitLab project's built-in registry using the `GITHUB_TOKEN` equivalent (`${CI_JOB_TOKEN}`), negating the need for managing separate Docker Hub credentials.

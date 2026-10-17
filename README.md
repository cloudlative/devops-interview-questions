# DevOps interview questions (with answers)

Free, scenario-based practice questions for DevOps, SRE and cloud interviews: Kubernetes, Docker, Azure, Google Cloud, Git, CI/CD and Istio. Each answer is hidden until you click, with a short explanation of *why*.

Want to test yourself properly? Every topic has a **free 10-question skills check** with explanations - no sign-up: [braintally.com/technical-assessments](https://braintally.com/technical-assessments)

Contributions welcome: found a wrong answer or have a better explanation? Open an issue or a pull request.

## Contents

- [Kubernetes](#kubernetes)
- [Docker](#docker)
- [Microsoft Azure](#microsoft-azure)
- [Google Cloud](#google-cloud)
- [Git](#git)
- [CI/CD and GitHub Actions](#cicd-and-github-actions)
- [Istio](#istio)

## Kubernetes

### Kubernetes Q1

A pod has been in Init:0/1 for twenty minutes. Its init container runs the loop below. kubectl get svc -n shop lists api and web only. Why does the pod not start?

```
initContainers:
- name: wait-for-db
  image: busybox:1.36
  command: ["sh", "-c", "until nslookup db; do sleep 2; done"]
```

- **A.** There is no Service named db in the namespace, so the lookup never succeeds and the init container never exits
- **B.** busybox cannot run nslookup
- **C.** Init containers time out after ten minutes
- **D.** The main container must start before the init container

<details><summary>Show answer</summary>

**A.** Regular containers start only after every init container exits successfully. The loop waits for a name that does not exist, so create the db Service (or fix the name) and the pod continues.

</details>

### Kubernetes Q2

A kubeadm cluster runs Kubernetes 1.29 and must reach 1.31. What is the supported path?

- **A.** Upgrade the worker nodes to 1.31 first, then the control plane
- **B.** Run kubeadm upgrade apply v1.31.x directly on the control plane
- **C.** Upgrade one minor version at a time: control plane to 1.30, then nodes to 1.30, then repeat for 1.31
- **D.** Reinstall the cluster on 1.31 and restore etcd from a 1.29 snapshot

<details><summary>Show answer</summary>

**C.** kubeadm cannot skip minor versions, and the control plane must be upgraded before the kubelets. The kubelet may lag the API server by up to three minor versions, but never run ahead of it.

</details>

### Kubernetes Q3

A StatefulSet db has three replicas behind a headless Service also called db. You scale it from 3 to 2. Which pod is removed, and what happens to its PersistentVolumeClaim by default?

- **A.** db-0 is removed and its PVC is deleted
- **B.** A random pod is removed and its PVC is kept
- **C.** db-2 is removed and its PVC is deleted with it
- **D.** db-2 is removed and its PVC is kept, so scaling back up reattaches the same data

<details><summary>Show answer</summary>

**D.** StatefulSets remove pods from the highest ordinal down. The default persistentVolumeClaimRetentionPolicy is Retain for both scale-down and deletion, so data-db-2 stays and is reused when db-2 returns.

</details>

### Kubernetes Q4

A Job has completions: 10 and parallelism: 3. Each pod processes one item and exits successfully. How does the Job run?

- **A.** It starts 10 pods at once
- **B.** It runs at most 3 pods at a time, starting new ones as others succeed, until 10 have completed
- **C.** It runs 3 pods once and then stops
- **D.** It runs 30 pods in total

<details><summary>Show answer</summary>

**B.** parallelism caps how many pods run concurrently and completions sets how many successful pods the Job needs. Failed pods are retried up to backoffLimit and do not count towards completions.

</details>


➡️ More Kubernetes questions: [free 10-question Kubernetes check](https://braintally.com/technical-assessments/kubernetes/check)


## Docker

### Docker Q1

The Dockerfile has stages deps, test and release. CI should run the unit tests by building up to the test stage only. Which command does that?

- **A.** docker build --stage test .
- **B.** docker build -f test .
- **C.** docker build --build-arg STAGE=test .
- **D.** docker build --target test .

<details><summary>Show answer</summary>

**D.** --target stops the build at the named stage, skipping later ones such as release. BuildKit also skips stages the target does not depend on.

</details>

### Docker Q2

A Compose file has a mail-catcher service you want only when debugging, not on a normal docker compose up. What is the intended way?

- **A.** Comment it out and uncomment it when needed
- **B.** Give the service profiles: [debug] and start it with docker compose --profile debug up
- **C.** Set replicas: 0 on the service
- **D.** Put it in a separate Compose project with a different name

<details><summary>Show answer</summary>

**B.** Services with a profile are skipped unless that profile is activated with --profile or COMPOSE_PROFILES. Services without profiles always start, so the normal stack is unchanged.

</details>

### Docker Q3

docker rmi app:1.0 prints Untagged: app:1.0, but docker system df shows no space freed. Why?

```
$ docker image ls
REPOSITORY   TAG      IMAGE ID       SIZE
app          latest   4f2c9e1d7a3b   850MB
```

- **A.** rmi only deletes the image after a daemon restart
- **B.** The image is still referenced by another tag, app:latest, which points to the same image ID; only the name was removed
- **C.** Images are stored in the build cache
- **D.** rmi needs --force to free space

<details><summary>Show answer</summary>

**B.** Tags are names for an image ID, and the layers are deleted only when no tag or container references the image any more. Removing the last tag (or the image by ID) frees the space.

</details>

### Docker Q4

A developer asks to be added to the docker group on a shared build server so they no longer need sudo. Why do security teams treat that request carefully?

- **A.** Members of the docker group can only see their own containers
- **B.** The group stops working after a reboot
- **C.** It disables image signing
- **D.** Access to the Docker daemon is equivalent to root on the host, because a member can start a privileged container that mounts the host's filesystem

<details><summary>Show answer</summary>

**D.** The daemon runs as root and does whatever its clients ask, such as docker run -v /:/host --privileged. Rootless Docker or a CI system that controls what is built limit that risk.

</details>


➡️ More Docker questions: [free 10-question Docker check](https://braintally.com/technical-assessments/docker/check)


## Microsoft Azure

### Microsoft Azure Q1

Your Bicep file deploys subnets into the virtual network core-vnet, which another team manages in the same resource group. You must refer to that VNet without redeploying or changing it. How do you declare it?

- **A.** resource vnet 'Microsoft.Network/virtualNetworks@2024-05-01' existing = { name: 'core-vnet' }
- **B.** Copy the full VNet definition into your file
- **C.** module vnet 'core-vnet.bicep' = { name: 'core-vnet' }
- **D.** param vnet object = reference('core-vnet')

<details><summary>Show answer</summary>

**A.** The existing keyword gives a symbolic reference to a resource that is already deployed, so you can use its properties or make it the parent of child resources without redeploying it. Copying the definition would redeploy and possibly overwrite the other team's settings.

</details>

### Microsoft Azure Q2

A Cosmos DB container uses /country as its partition key. Most traffic comes from one country, and requests fail with 429 although total RU usage is well below what you provisioned. Why?

- **A.** The account is in the wrong region
- **B.** 429 means the item is too large
- **C.** The indexing policy is set to none
- **D.** Throughput is split across physical partitions, so one hot partition key value runs out of its share; choose a high-cardinality key such as /customerId

<details><summary>Show answer</summary>

**D.** Provisioned RU/s is divided among physical partitions, and a single logical partition cannot use more than its partition's share. A key with many evenly used values spreads the load.

</details>

### Microsoft Azure Q3

Security wants nobody to hold the Global Administrator role permanently. Admins should activate it for a few hours, with a justification and approval, only when needed. What provides that?

- **A.** Microsoft Entra Privileged Identity Management, with eligible role assignments
- **B.** An Azure Policy that denies role assignments
- **C.** A Conditional Access policy that blocks admin sign-ins
- **D.** A custom RBAC role with a time limit

<details><summary>Show answer</summary>

**A.** PIM makes users eligible for a role and lets them activate it just in time, with time limits, MFA, justification and approval. RBAC roles themselves have no expiry, and Conditional Access cannot grant roles.

</details>

### Microsoft Azure Q4

Resizing a running VM to a much larger size fails, saying the size is not available in the current hardware cluster, although the size is offered in the region. What do you do?

- **A.** Stop (deallocate) the VM, then resize it, so it can be placed on hardware that offers the new size
- **B.** Delete the VM and recreate it from scratch
- **C.** Open a quota request for the region
- **D.** Detach the data disks first

<details><summary>Show answer</summary>

**A.** A running VM can only be resized to sizes its current hardware cluster supports. After deallocation Azure can place it anywhere in the region, so more sizes become available; the restart is the cost.

</details>


➡️ More Microsoft Azure questions: [free 10-question Microsoft Azure check](https://braintally.com/technical-assessments/microsoft-azure/check)


## Google Cloud

### Google Cloud Q1

A Cloud Run service mounts the secret api-key from Secret Manager, but new revisions fail to deploy with a permission error on the secret. What is missing?

- **A.** roles/secretmanager.admin for the developer
- **B.** roles/secretmanager.secretAccessor on the secret for the service's runtime service account
- **C.** The secret must be in the same region as the service
- **D.** The secret must be stored as an environment variable first

<details><summary>Show answer</summary>

**B.** Cloud Run reads the secret as the service's identity, so that service account needs Secret Manager Secret Accessor. Secrets are global by default, and admin rights for the deployer do not help the runtime identity.

</details>

### Google Cloud Q2

On a GKE Standard cluster, new pods stay Pending with Insufficient memory during traffic peaks, though the node pool has room in its quota. How do you let the cluster add nodes automatically?

- **A.** Add a HorizontalPodAutoscaler
- **B.** Increase the pods' memory limits
- **C.** Enable the cluster autoscaler on the node pool with minimum and maximum node counts
- **D.** Switch the cluster to a regional control plane

<details><summary>Show answer</summary>

**C.** The cluster autoscaler adds nodes when pods cannot be scheduled and removes underused ones. An HPA adds pods, which would only create more Pending pods, and larger limits make scheduling harder.

</details>

### Google Cloud Q3

Whenever a file is uploaded to the bucket incoming-scans, a Cloud Run service should process it automatically. Which managed way delivers those events to the service?

- **A.** A cron job that lists the bucket every minute
- **B.** A Cloud Storage lifecycle rule
- **C.** An Eventarc trigger on the object finalized event for that bucket, targeting the Cloud Run service
- **D.** VPC flow logs

<details><summary>Show answer</summary>

**C.** Eventarc routes events such as google.cloud.storage.object.v1.finalized to Cloud Run, with authentication handled by a trigger service account. Polling adds delay and cost, and lifecycle rules only delete or change storage class.

</details>

### Google Cloud Q4

Application logs older than 30 days have disappeared from Logs Explorer, but compliance requires keeping them for a year. What do you change?

- **A.** Increase the retention of the _Default log bucket (or route these logs to a bucket with 365-day retention)
- **B.** Enable Data Access audit logs
- **C.** Raise the logging API quota
- **D.** Nothing; logs cannot be kept longer than 30 days

<details><summary>Show answer</summary>

**A.** Logs routed to the _Default bucket are kept for 30 days unless its retention is changed, which can be set up to 3650 days at extra cost. A sink to a dedicated bucket, BigQuery or Cloud Storage also works for long-term storage.

</details>


➡️ More Google Cloud questions: [free 10-question Google Cloud check](https://braintally.com/technical-assessments/google-cloud/check)


## Git

### Git Q1

A function's lines 40 to 60 in billing.py look wrong, and you want to know which commit and author last changed each of those lines. Which command?

- **A.** git log billing.py
- **B.** git blame -L 40,60 billing.py
- **C.** git diff HEAD~10 billing.py
- **D.** git show billing.py

<details><summary>Show answer</summary>

**B.** blame shows, line by line, the commit, author and date that last touched each line, and -L limits it to a range. git log lists commits for the file but not which line came from where.

</details>

### Git Q2

A test passes on tag v3.0 and fails on main, with 400 commits in between. What is the fastest systematic way to find the commit that broke it?

- **A.** git log -S test
- **B.** git blame on the test file
- **C.** git bisect start main v3.0, then git bisect run ./run-test.sh
- **D.** Revert commits one at a time from the newest

<details><summary>Show answer</summary>

**C.** bisect does a binary search between a bad and a good commit, so about nine test runs find the culprit among 400; bisect run automates it with a script's exit code. blame shows who last edited lines, not what caused the failure.

</details>

### Git Q3

Designers commit 200 MB .psd files, and every clone of the repository has become very slow. Which approach keeps the files versioned without bloating the history?

- **A.** Compress the files with zip before committing
- **B.** Add *.psd to .gitignore
- **C.** Use Git LFS (git lfs track "*.psd") so the repository stores small pointers and the files live in LFS storage
- **D.** Use git gc --aggressive

<details><summary>Show answer</summary>

**C.** Git LFS replaces large binaries with pointer files and downloads the real content on checkout. Ignoring them stops versioning, and gc cannot shrink binaries that are already in history; those need a history rewrite to remove.

</details>

### Git Q4

The team keeps a pre-commit hook in .githooks/pre-commit in the repository, but it never runs for anyone after cloning. Why, and what fixes it?

- **A.** Hooks only run on the server
- **B.** The hook must be named pre-commit.sh
- **C.** Hooks require git commit --verify
- **D.** Git only runs hooks from .git/hooks, which is not versioned; set git config core.hooksPath .githooks (and make the file executable)

<details><summary>Show answer</summary>

**D.** Hooks are not cloned, for safety, and Git looks in .git/hooks unless core.hooksPath points elsewhere. Each developer (or a setup script) sets hooksPath to the versioned folder.

</details>


➡️ More Git questions: [free 10-question Git check](https://braintally.com/technical-assessments/git/check)


## CI/CD and GitHub Actions

### CI/CD and GitHub Actions Q1

A job that runs end-to-end tests hung on a stuck browser and kept the runner busy for six hours, using up CI minutes. How do you stop that happening again?

- **A.** Set timeout-minutes on the job (for example 30), well below the 6-hour default
- **B.** Use continue-on-error: true
- **C.** Add if: success()
- **D.** Run the job on a larger runner

<details><summary>Show answer</summary>

**A.** Jobs on GitHub-hosted runners default to a 360-minute limit. A job-level (or step-level) timeout-minutes cancels a hung run early; continue-on-error only changes how a failure is reported.

</details>

### CI/CD and GitHub Actions Q2

A team wants to ship a new version to 5 percent of users, watch error rates, and expand gradually or roll back quickly. Which deployment strategy is this?

- **A.** Recreate
- **B.** Canary
- **C.** Blue-green with an instant full switch
- **D.** Rolling update with maxUnavailable: 100%

<details><summary>Show answer</summary>

**B.** A canary release sends a small share of real traffic to the new version and widens it as metrics stay healthy. Blue-green switches all traffic at once between two full environments, and recreate has downtime.

</details>

### CI/CD and GitHub Actions Q3

A team is choosing between a CI job that runs kubectl apply against production and a pull-based GitOps agent (such as Argo CD or Flux) running in the cluster. What is a key security advantage of the pull-based agent?

- **A.** It makes builds faster
- **B.** It removes the need for a container registry
- **C.** It encrypts Kubernetes Secrets in etcd
- **D.** CI no longer needs credentials to the cluster; the agent inside the cluster pulls the desired state from Git and applies it

<details><summary>Show answer</summary>

**D.** With push deployments every CI runner holds a powerful kubeconfig, which becomes a target. A pull-based agent keeps cluster credentials inside the cluster and only needs read access to Git, and it also corrects drift.

</details>

### CI/CD and GitHub Actions Q4

A Slack step should post only when an earlier step in the job fails. With no condition it never runs on failure. Which condition does it need?

- **A.** if: failure()
- **B.** if: always()
- **C.** if: success() == false
- **D.** if: ${{ job.status }} == 'failed'

<details><summary>Show answer</summary>

**A.** Steps run only after success by default. failure() runs the step when any previous step failed; always() would also post on success and even after cancellation.

</details>


➡️ More CI/CD and GitHub Actions questions: [free 10-question CI/CD and GitHub Actions check](https://braintally.com/technical-assessments/ci-cd-and-github-actions/check)


## Istio

### Istio Q1

To upgrade Istio with less risk, you install the new control plane next to the old one and move namespaces over one at a time. Which mechanism supports that?

- **A.** Running two istio-system namespaces
- **B.** istioctl upgrade --force on one namespace
- **C.** Changing the image tag of the istio-proxy containers
- **D.** A revision-based (canary) upgrade: install with --set revision=1-24-0, relabel each namespace with istio.io/rev=1-24-0 and restart its pods

<details><summary>Show answer</summary>

**D.** Revisions let two control planes run side by side, and the istio.io/rev label chooses which one injects or manages a namespace's workloads. Once everything has moved, the old revision is removed.

</details>

### Istio Q2

Teams want Envoy access logs for every workload in the mesh. What is the current, supported way to turn them on mesh-wide?

- **A.** Set --log_output_level=debug on istiod
- **B.** An EnvoyFilter on every Deployment
- **C.** A Telemetry resource in the root namespace with accessLogging using the envoy provider
- **D.** kubectl logs --all-containers on the proxies

<details><summary>Show answer</summary>

**C.** The Telemetry API configures access logging, metrics and tracing per mesh, namespace or workload, and a resource in the root namespace applies everywhere. EnvoyFilter is low-level and fragile, and istiod's log level does not affect proxies.

</details>

### Istio Q3

Your namespace runs in ambient mode. You want HTTP-level routing and authorization (by path and method) for the reviews service. What do you deploy?

- **A.** A waypoint proxy, for example istioctl waypoint apply -n shop --enroll-namespace
- **B.** A sidecar inside the reviews pods
- **C.** Nothing: ztunnel handles HTTP rules
- **D.** An extra ingress gateway

<details><summary>Show answer</summary>

**A.** In ambient mode ztunnel provides L4 features (mTLS, L4 authorization, telemetry), and L7 features need a waypoint proxy. --enroll-namespace labels the namespace so its services use the waypoint.

</details>

### Istio Q4

Before installing or upgrading Istio on a cluster, you want a quick check for problems such as unsupported Kubernetes versions or conflicting settings. Which command?

- **A.** istioctl proxy-status
- **B.** kubectl get crds | grep istio
- **C.** istioctl x precheck
- **D.** istioctl dashboard kiali

<details><summary>Show answer</summary>

**C.** precheck inspects the cluster for issues that would affect an install or upgrade, such as version support and existing configuration. proxy-status checks running proxies, which only exist after installation.

</details>


➡️ More Istio questions: [free 10-question Istio check](https://braintally.com/technical-assessments/istio/check)


---

Questions are written and maintained by [BrainTally](https://braintally.com). Licensed CC BY 4.0 - share and adapt with credit.

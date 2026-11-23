# jenkins-agent-python-312
Provides a container image of the Python **3.12.16** runtime for use as a Jenkins agent.
It matches the Python version the apps run on (`.python-version`, `python:3.12.16-slim-bookworm`).

* Base image: `ghcr.io/redhat-cop/containers-quickstarts/jenkins-agent:v4.18` (RHEL 8.10, OpenJDK 21).
* Python 3.12.16 is compiled from the python.org source tarball (SHA256-verified) in a builder
  stage on the same base image, installed under `/usr/local`. The UBI8 `python3.12` RPM is 3.12.15,
  so it is not used. `python3`, `python`, `pip3`, and `pip` all resolve to 3.12.16.
* Included tools: `git`, `curl`, `tar`, `gzip`, `jq` 1.7.1, `yq` v4.50.1, `grep -P`, `oc`, `kubectl`, and `twine`.
* The image runs as an arbitrary UID in group 0 (OpenShift). `/home/jenkins` is group-0 writable,
  and `/home/jenkins/.local/bin` is on `PATH`, so `pip install --user <tool>` works.
  A plain `pip install` falls back to a user install automatically.
* App-specific tools (flake8, ruff, …) are **not** baked in; pipelines pin and install them.

To change the Python version, bump `PYTHON_VERSION` **and** `PYTHON_SHA256` in the `Dockerfile`, then bump `version.json`.
The SHA256 of the `.tar.xz` comes from the python.org release API:
```bash
V=3.12.16
REL=$(curl -s "https://www.python.org/api/v2/downloads/release/?name=Python%20$V" | jq -r '.[0].resource_uri' | awk -F/ '{print $(NF-1)}')
curl -s "https://www.python.org/api/v2/downloads/release_file/?release=$REL" | jq -r '.[] | select(.url|endswith(".tar.xz")) | .sha256_sum'
```

## Build local
```bash
docker build -t jenkins-agent-python-312:v1.0.0 jenkins-agents/jenkins-agent-python-312
```
(`podman build` works the same way.) The build compiles CPython with PGO and LTO. Expect about 10 minutes on a 16-core machine.

## Run local
Run it the way OpenShift does, as a random UID in group 0:
```bash
docker run --rm -it --user 1000870000:0 --entrypoint bash jenkins-agent-python-312:v1.0.0
```

## Publish to the cluster
Jenkins (OpenShift sync plugin) turns every ImageStream labelled `role=jenkins-slave` in `labs-ci-cd`
into a Kubernetes pod template. The template name, and so its Jenkins label, is the ImageStream name:
`jenkins-agent-python-312`. Pick **one** of the routes below.

### A. Build in-cluster (preferred: no local push, no registry credentials)
`.openshift/templates/jenkins-agent-generic-template.yml` creates the ImageStream (already labelled
`role: jenkins-slave`) and a Docker-strategy BuildConfig.

The template sets `dockerStrategy.from` to `BUILDER_IMAGE_NAME`. On a multi-stage Dockerfile, OpenShift
replaces **only the last `FROM`** with that image. The builder stages keep the image named in the Dockerfile.
This Dockerfile compiles Python in a builder stage and copies it into the final stage, so both stages
must be the same OS. The override breaks that in two ways:
* Leave `BUILDER_IMAGE_NAME` at its default (`openshift3/jenkins-slave-base-rhel7`) and the final stage
  becomes RHEL 7. The build fails, or the image ships a Python that can't load its libraries.
* Pass the matching image today and the build works. But when someone later bumps the base in the
  Dockerfile, the BuildConfig silently keeps the final stage on the old image.

So remove the override (and the `ImageChange` trigger, which never fires for a `DockerImage` source anyway)
before applying. The Dockerfile then stays the single source of truth:

```bash
oc login --server=https://api.develop.lxp.academy.who.int:6443 ...   # your usual login
oc project labs-ci-cd

oc process -f .openshift/templates/jenkins-agent-generic-template.yml \
  -p NAME=jenkins-agent-python-312 \
  -p SOURCE_REPOSITORY_REF=master \
  -p SOURCE_CONTEXT_DIR=jenkins-agents/jenkins-agent-python-312 \
  -o json \
| jq '(.items[] | select(.kind=="BuildConfig") | .spec) |= (del(.strategy.dockerStrategy.from) | .triggers |= map(select(.type != "ImageChange")))' \
| oc apply -f -

# The ConfigChange trigger starts the first build as soon as the BuildConfig is created; follow it:
oc logs -f bc/jenkins-agent-python-312
```
Set `SOURCE_REPOSITORY_REF` to the branch or tag to build from (e.g. `feat-python-3.12.16` before merge).

If the BuildConfig was already created with the override, patch it instead:
```bash
oc patch bc/jenkins-agent-python-312 -n labs-ci-cd --type=json \
  -p '[{"op":"remove","path":"/spec/strategy/dockerStrategy/from"}]'
```

Rebuild after changes are pushed to the branch/tag:
```bash
oc start-build jenkins-agent-python-312 -n labs-ci-cd --follow
```
To build from a different ref without re-applying, run
`oc patch bc/jenkins-agent-python-312 -n labs-ci-cd --type=merge -p '{"spec":{"source":{"git":{"ref":"<ref>"}}}}'`
and then `oc start-build`.

Tag a release (keep `latest` for Jenkins, add an immutable version tag for rollback):
```bash
oc tag labs-ci-cd/jenkins-agent-python-312:latest labs-ci-cd/jenkins-agent-python-312:v1.0.0
```

### B. Build locally and push

#### B1. To the OpenShift internal registry (default route)
```bash
oc login ... && oc project labs-ci-cd
REGISTRY=$(oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}')
# If that route doesn't exist, a cluster-admin must expose it:
#   oc patch configs.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"defaultRoute":true}}'

podman login -u "$(oc whoami)" -p "$(oc whoami -t)" "$REGISTRY"   # add --tls-verify=false if the route uses a self-signed cert

podman build --platform linux/amd64 -t jenkins-agent-python-312:v1.0.0 jenkins-agents/jenkins-agent-python-312
podman tag jenkins-agent-python-312:v1.0.0 "$REGISTRY/labs-ci-cd/jenkins-agent-python-312:v1.0.0"
podman tag jenkins-agent-python-312:v1.0.0 "$REGISTRY/labs-ci-cd/jenkins-agent-python-312:latest"
podman push "$REGISTRY/labs-ci-cd/jenkins-agent-python-312:v1.0.0"
podman push "$REGISTRY/labs-ci-cd/jenkins-agent-python-312:latest"

# Pushing creates the ImageStream; label it so Jenkins picks it up:
oc label is/jenkins-agent-python-312 role=jenkins-slave -n labs-ci-cd --overwrite
```

#### B2. To ACR (`whoalxp.azurecr.io`) and import into the cluster
```bash
az acr login -n whoalxp      # or: podman login whoalxp.azurecr.io

podman build --platform linux/amd64 -t whoalxp.azurecr.io/jenkins-agent-python-312:v1.0.0 jenkins-agents/jenkins-agent-python-312
podman tag whoalxp.azurecr.io/jenkins-agent-python-312:v1.0.0 whoalxp.azurecr.io/jenkins-agent-python-312:latest
podman push whoalxp.azurecr.io/jenkins-agent-python-312:v1.0.0
podman push whoalxp.azurecr.io/jenkins-agent-python-312:latest
```
`labs-ci-cd` needs a pull secret for ACR. Skip this if one already exists: `oc get secrets -n labs-ci-cd | grep azurecr`.
```bash
oc create secret docker-registry acr-pull -n labs-ci-cd \
  --docker-server=whoalxp.azurecr.io \
  --docker-username=<acr-token-or-sp-id> --docker-password=<password>
oc secrets link default acr-pull --for=pull -n labs-ci-cd
oc secrets link jenkins acr-pull --for=pull -n labs-ci-cd
```
Import into an ImageStream and label it:
```bash
oc import-image jenkins-agent-python-312:latest \
  --from=whoalxp.azurecr.io/jenkins-agent-python-312:latest \
  --confirm --scheduled --reference-policy=local -n labs-ci-cd
oc tag labs-ci-cd/jenkins-agent-python-312:latest labs-ci-cd/jenkins-agent-python-312:v1.0.0
oc label is/jenkins-agent-python-312 role=jenkins-slave -n labs-ci-cd --overwrite
```
`--scheduled` re-imports `latest` periodically, so new ACR pushes reach the cluster without re-running the import.
`--reference-policy=local` makes pods pull through the internal registry, which holds the ACR credentials.

## Jenkins
1. Check that a pod template named `jenkins-agent-python-312` appears under
   **Manage Jenkins → Clouds → openshift** (or `kubernetes`) → **Pod Templates**.
   The sync plugin picks up the labelled ImageStream within a minute or so.
2. If it doesn't appear, add a pod template by hand:
   * Name and Labels: `jenkins-agent-python-312`
   * Container name: `jnlp`
   * Docker image: `image-registry.openshift-image-registry.svc:5000/labs-ci-cd/jenkins-agent-python-312:latest`
   * Always pull image: checked (pull policy `Always`)
   * Service account: `jenkins`
   * Arguments to pass to the command: `${computer.jnlpmac} ${computer.name}`, with "Command to run" left empty
3. Prove it with a throwaway Pipeline job whose script is [`Jenkinsfile.test`](Jenkinsfile.test).
   The job should print `Python 3.12.16`, pip, Java, and `flake8 --version`.
4. Switch a pipeline over by changing its agent label:
   ```groovy
   agent { node { label "jenkins-agent-python-312" } }
   ```

**Rollback:** change the pipeline's label back to `jenkins-agent-python`. The existing `jenkins-agent-python`
and `jenkins-agent-python-39` images are untouched. To roll back the agent image itself, run
`oc tag labs-ci-cd/jenkins-agent-python-312:v1.0.0 labs-ci-cd/jenkins-agent-python-312:latest`.

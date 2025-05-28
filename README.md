# ${{ values.name }}

A demo of best DevSecOps practices with focus on automation, collaboration, security and documentation.

## Requirements

1. [go v1.24+](https://go.dev/doc/install)
2. [git cli](https://git-scm.com/downloads)
3. [make](https://www.gnu.org/software/make/) - Optional. In case of Windows OS, it is possible to execute the build commands defined in the Makefile from the command line.

4. [Kind Kubernetes](https://kind.sigs.k8s.io/) 

## Build hello application

Clone hello repository. 

```sh
> git clone git@github.com:altimetrik-digital-enablement-demo-hub/${{ values.name }}.git
> cd ${{ values.name }}
```

Build the binary. TAG env variable sets the application version. If not specified, the default is `dev`

```sh
> TAG=0.0.16 make build
Build availalbe at ./bin/${{ values.name }}

> ./bin/${{ values.name }} version
hello version
  version: 0.0.16
  commit: 1de9255
```

## Build hello container image

```sh
$ make docker-build
docker build \
	--build-arg VERSION="0.0.16" \
	--build-arg COMMIT="1de9255" \
	--build-arg BUILD_UNIX_TIME="174645213735" \
	-f docker/Dockerfile \
	 -t ${{ values.owner }}/${{ values.name }}:0.0.16 .
[+] Building 13.2s (14/14) FINISHED
 => [internal] load build definition from Dockerfile
 ...
 => => naming to docker.io/${{ values.owner }}/${{ values.name }}:0 
 ....
 => => unpacking to docker.io/${{ values.owner }}/${{ values.name }}:0.0.16
```

Test the new container image
```sh
$ docker run --rm -ti ${{ values.owner }}/${{ values.name }}:0.0.16 /hello version
hello version
  version: 0.0.16
  commit: 1de9255
```

## Kubernetes deployment

### Kind K8S cluster

kind is a tool for running local Kubernetes clusters using Docker container “nodes”.

Create a local `hello` kind cluster.

```sh
$ kind create cluster --name hello
$ kind get clusters
hello
```

Upload the dsocker image to the `hello` kind cluster.

```sh
$ make kind-docker-upload

# or, manually
$ kind --name hello load docker-image ${{ values.owner }}/${{ values.name }}:0.0.16 
```

Ensure `hello` container image is set to correct version in `deploy/manifests/deployment.yaml` manifest file.

```yaml
 spec:
    serviceAccountName: hello
    containers:
    - name: hello
      image: ${{ values.owner }}/${{ values.name }}:0.0.16
```

Create `hello` namespace

```sh
$ kubectl create ns hello
namespace/hello created
```

Deploy `hello` app in the newly created namespace

```sh
$ kubectl create -f deploy/manifests
deployment.apps/hello created
serviceaccount/hello created
service/hello created

$ kubectl get all -n hello
NAME                     READY   STATUS    RESTARTS   AGE
kubectl get all -n hello
NAME                         READY   STATUS    RESTARTS   AGE
pod/hello-69c9cf5648-92hc9   1/1     Running   0          52s

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/hello   ClusterIP   10.96.28.147   <none>        80/TCP    52s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello   1/1     1            1           53s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-69c9cf5648   1         1         1       52s
```


### Debugging hello Pod

```sh
kubectl -n hello \
debug -it \
--container=debug-container \
--image=alpine \
--target=hello \
hello-69c9cf5648-92hc9

Targeting container "hello". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
--profile=legacy is deprecated and will be removed in the future. It is recommended to explicitly specify a profile, for example "--profile=general".
If you don't see a command prompt, try pressing enter.
/ # ps axww
PID   USER     TIME  COMMAND
    1 root      0:00 /hello server
   16 root      0:00 /bin/sh
   26 root      0:00 ps axww
/ #
/ # apk add curl
/ # apk add jq
/ # curl -s localhost:8080/hello | jq
{
  "message": "Hello, World!"
}
/ # curl -s localhost:8080/version | jq
{
  "COMMIT": "1de9255",
  "DATE": "174641174022",
  "VERSION": "0.0.16"
}
```

## CI build and release workflows

### build

This workflow builds the binary, sets the applicatin version and commit hash, and if the branch name is main it creates a git tag using the applicatin version, followed by a dispatch call to the release workflow.

The version is based on the branch type:
1. main - vN.N.N; It keeps the SemVer version from 
2. other branch - The version follows the following format: `<branchname>-<YYYYMMDD>.<github_run_number>`

### release

Release is invoked by a push tag event. The tag value has to have the SemVer format with three parts: `v*.*.*`
The release is automatically called from the build  workflow using a dispatch event call after the build workflow pushes a tag to the main branch.

## Git Hooks scripts

The use of git hooks has been documented in [doc/git-hooks.md](doc/git-hooks.md) file.

## Git commands

Git commands are implemented in NuShell. They are wrappers around regular git commands tha provide additional functionality.

### Git Squash

Usage TODO

# Semantic Releases

TODO

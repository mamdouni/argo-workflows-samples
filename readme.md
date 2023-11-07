# Argo Workflows Example

To install argo on your local machine, start by installing minikube first.

Then you can follow this article :
- https://argoproj.github.io/argo-workflows/quick-start/

You have to set-up :
- Argo server
- Argo CLI client

In my case, i'm using macos and the steps was :

```shell
# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.5.1/argo-darwin-amd64.gz

# Unzip
gunzip argo-darwin-amd64.gz

# Make binary executable
chmod +x argo-darwin-amd64

# Move binary to path
mv ./argo-darwin-amd64 /usr/local/bin/argo

# Test installation
argo version
```

here's the result :
```shell
argo version
```

```log
argo: v3.5.1
  BuildDate: 2023-11-03T19:47:06Z
  GitCommit: 877c5523066e17687856fe3484c9b2d398e986f5
  GitTreeState: clean
  GitTag: v3.5.1
  GoVersion: go1.21.3
  Compiler: gc
  Platform: darwin/amd64
```

After disabling the authentication as mentioned in the docs, argo server was accessible via this url :
- https://localhost:2746/workflows?limit=50

and to get access, don't forget this one :

```shell
kubectl -n argo port-forward deployment/argo-server 2746:2746
```

- https://localhost:2746/workflows/argo?limit=50

## Features

Some features will not be presented in this repo but they will be mentionned here as they are important to know.

### Database

If you want to keep completed workflows for a long time, you can use the workflow archive to save them in a Postgres or MySQL database.
- https://argoproj.github.io/argo-workflows/workflow-archive/

## References
- https://argoproj.github.io/argo-workflows/quick-start/
- https://argoproj.github.io/argo-workflows/workflow-concepts/
- https://argoproj.github.io/argo-workflows/walk-through/argo-cli/
- https://www.youtube.com/watch?v=c3qJr6L8nHg

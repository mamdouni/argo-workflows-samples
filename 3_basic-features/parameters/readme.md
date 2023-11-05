# Parameters

In this section, we will check some features of passing parameters.

Here, we will define an input argument for our workflow :

```yaml
  
  entrypoint: whalesay
  
  # Define some hard-coded arguments to pass via argo but you can remove this part and pass you argument through command line
  arguments:
    parameters:
    - name: message
      value: hello world

  templates:
  - name: whalesay
    inputs:
      parameters:
      - name: message       # parameter declaration
    container:
      # run cowsay with that message input parameter as args
      image: busybox
      command: [echo]
      args: ["{{inputs.parameters.message}}"]
```

## Using the template

```shell
argo submit --watch parameters.yaml
```

```log
Name:                hello-world-parameters-htrtg
Namespace:           argo
ServiceAccount:      unset (will run with the default ServiceAccount)
Status:              Succeeded
Conditions:
 PodRunning          False
 Completed           True
Created:             Sun Nov 05 20:37:58 +0100 (10 seconds ago)
Started:             Sun Nov 05 20:37:58 +0100 (10 seconds ago)
Finished:            Sun Nov 05 20:38:08 +0100 (now)
Duration:            10 seconds
Progress:            1/1
ResourcesDuration:   5s*(1 cpu),5s*(100Mi memory)
Parameters:
  message:           hello world

STEP                             TEMPLATE  PODNAME                       DURATION  MESSAGE
 ✔ hello-world-parameters-htrtg  whalesay  hello-world-parameters-htrtg  5s
```

Check the parameters section this time.

```shell
argo logs @latest
```

```log
hello-world-parameters-htrtg: hello world
hello-world-parameters-htrtg: time="2023-11-05T19:38:03.590Z" level=info msg="sub-process exited" argo=true error="<nil>"
```

## Using command line


```shell
argo submit --watch parameters.yaml -p message="goodbye world"
```

```log
Name:                hello-world-parameters-lcx4h
Namespace:           argo
ServiceAccount:      unset (will run with the default ServiceAccount)
Status:              Succeeded
Conditions:
 PodRunning          False
 Completed           True
Created:             Sun Nov 05 20:41:01 +0100 (10 seconds ago)
Started:             Sun Nov 05 20:41:01 +0100 (10 seconds ago)
Finished:            Sun Nov 05 20:41:11 +0100 (now)
Duration:            10 seconds
Progress:            1/1
ResourcesDuration:   5s*(1 cpu),5s*(100Mi memory)
Parameters:
  message:           goodbye world

STEP                             TEMPLATE  PODNAME                       DURATION  MESSAGE
 ✔ hello-world-parameters-lcx4h  whalesay  hello-world-parameters-lcx4h  6s
```

Check the parameters section.

```shell
argo logs @latest
```

```log
hello-world-parameters-lcx4h: goodbye world
hello-world-parameters-lcx4h: time="2023-11-05T19:41:06.798Z" level=info msg="sub-process exited" argo=true error="<nil>"
```

Let's check the UI :
> Go to the executed workflow and click on the unique step
![inputs](images/inputs.png)

You can also use a config file like this :

```shell
argo submit arguments-parameters.yaml --parameter-file params.yaml
```

Or even override the entrypoint :

```shell
argo submit arguments-parameters.yaml --entrypoint whalesay-caps
```

## Global parameters

As you can see from the last example :

```yaml
  templates:
  - name: whalesay
    inputs:
      parameters:
      - name: message       # parameter declaration
    container:
      # run cowsay with that message input parameter as args
      image: busybox
      command: [echo]
      args: ["{{inputs.parameters.message}}"]
```

Each time you need to use the parameter, you have to declare it first in the step before using it which is not handy.

This part exactly :

```yaml
    inputs:
      parameters:
      - name: message       # parameter declaration
```

The values set in the spec.arguments.parameters are globally scoped and can be accessed via {{workflow.parameters.parameter_name}}. This can be useful to pass information to multiple steps in a workflow. For example, if you wanted to run your workflows with different logging levels that are set in the environment of each container, you could have a YAML file similar to this one:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: global-parameters-
spec:
  entrypoint: A
  arguments:
    parameters:
    - name: log-level
      value: INFO

  templates:
  - name: A
    container:
      image: containerA
      env:
      - name: LOG_LEVEL
        value: "{{workflow.parameters.log-level}}"
      command: [runA]
  - name: B
    container:
      image: containerB
      env:
      - name: LOG_LEVEL
        value: "{{workflow.parameters.log-level}}"
      command: [runB]
```

So as you can see, the parameters part is declared globally in the workflow and can be accessed by any workflow you want.


```shell
argo submit --watch parameters-global.yaml
```

```log
Name:                hello-world-global-parameters-6cv8w
Namespace:           argo
ServiceAccount:      unset (will run with the default ServiceAccount)
Status:              Succeeded
Conditions:
 PodRunning          False
 Completed           True
Created:             Sun Nov 05 21:09:22 +0100 (9 seconds ago)
Started:             Sun Nov 05 21:09:22 +0100 (9 seconds ago)
Finished:            Sun Nov 05 21:09:32 +0100 (now)
Duration:            10 seconds
Progress:            1/1
ResourcesDuration:   5s*(1 cpu),5s*(100Mi memory)
Parameters:
  log-level:         WARN

STEP                                    TEMPLATE  PODNAME                              DURATION  MESSAGE
 ✔ hello-world-global-parameters-6cv8w  whalesay  hello-world-global-parameters-6cv8w  4s
```

Check the parameters section this time.

```shell
argo logs @latest
```

```log
hello-world-global-parameters-vc9h8: ARGO_PROGRESS_FILE_TICK_DURATION=3s
hello-world-global-parameters-vc9h8: LOG_LEVEL=WARN
hello-world-global-parameters-vc9h8: ARGO_SERVER_SERVICE_PORT=2746
```

## References
- https://argoproj.github.io/argo-workflows/walk-through/parameters/
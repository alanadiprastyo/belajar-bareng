## Create and run a basic task

1. Create and apply task
```
oc apply -f hello-world.yaml
```

2. create and apply taskrun
```
oc apply -f hello-world-run.yaml
```

3. verify taskrun
```
oc get taskrun hello-task-run
```

4. verify log task
```
oc logs --selector=tekton.dev/taskRun=hello-task-ru
```

## Create and run a basic pipeline

1. create and apply a task
```
oc apply -f goodbye-world.yaml
```

2. Create and apply a pipeline
```
oc apply -f hello-goodbye-pipeline.yaml
```

3. Running pipeline with pipelinrun
```
oc apply -f hello-goodbye-pipeline-run.yaml
```

4. show log the pipelinerun
```
tkn pipelinerun logs hello-goodbye-run -f
```

# Advanced Tekton Configuration
## Tekton Task

A Task is a collection of Steps that you define and arrange in a specific order of execution as part of your continuous integration flow. A Task executes as a Pod on your Kubernetes cluster. A Task is available within a specific namespace, while a ClusterTask is available across the entire cluster.

1. Apply source-lister.yaml
```
oc apply -f source-lister.yaml -n maung-project
```
2. verify task
```
tkn task ls
```
3. describe task
```
tkn task describe source-lister
```
4. running the task
```
tkn task start source-lister --showlog
```
Note: make sure contextDir use `quarkus`

Remember a Task is running as a pod therefore you can watch your pod to see task lifecycle: Init, PodInitializing, Running, and finally Completed as seen in the following listing:

## Cluster Task

- create new project
```
oc new-project maung-clustertask
```
- verify task
```
tkn task ls
```
note: no tasks found

- apply cluster task
```
oc apply -f clustertask-demo.yaml
```
- verify cluster task
```
tkn clustertask ls
```
- describe cluster task
```
tkn clustertask describe echoer
```
- run cluster task
```
tkn clustertask start echoer --showlog
```
- run cluster task in another project
```
oc project maung-project
tkn clustertask start echoer --showlog
```
### Points to ponder

- Tasks are namespace bound i.e. available only in namespace(s) where they were created
- Tasks resources are interacted using tkn task command and its options
- ClusterTasks are available across the cluster i.e. in any namesapce(s) of the cluster
- ClusterTasks resources are interacted using tkn clustertask command and its options

## Build Cloud Native Apps

Lets take an example of building a Cloud Native Java Application, at minimum the Task need to do the following:

- Clone the application from git.
- Build the Application from sources using build tool like Apache Maven or Gradle
- Build and push the container image to remote container registry like Quay.io or Docker Hub


```
oc project maung-project
oc apply -f build-app-task.yaml
tkn task ls
```
run task
```
tkn task start -n maung-project build-app \
  --param contextDir='springboot' \
  --param destinationImage='image-registry.openshift-image-registry.svc:5000/maung-project/tekton-tutorial-greeter' \
  --param storageDriver='vfs' \
  --showlog
```

## Pipeline
### Add task from catalog
```
oc apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/task/openshift-client/0.2/openshift-client.yaml -n maung-project
tkn task ls
```
### Create a pipeline
```
oc apply -f svc-deploy.yaml
tkn pipeline ls
```

### Run pipeline
```
oc apply -f pipeline-sa-role.yaml
tkn pipeline start -n maung-project svc-deploy \
  --param contextDir='springboot' \
  --param destinationImage='image-registry.openshift-image-registry.svc:5000/maung-project/tekton-tutorial-greeter' \
  --showlog
```
Note: error `fuse-overlayfs: cannot mount: No such file or directory` --> not yet workaround

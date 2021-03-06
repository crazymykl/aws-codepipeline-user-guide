# CodePipeline Concepts<a name="concepts"></a>

Modeling and configuring your automated release process is easier if you understand the concepts and terms used in AWS CodePipeline\. Here are some concepts to know about as you use CodePipeline\.

For an example of a DevOps pipeline, see [DevOps Pipeline Example](concepts-devops-example.md)\.

**Topics**
+ [Pipeline Terms](#concepts-pipeline-terms)

## Pipeline Terms<a name="concepts-pipeline-terms"></a>

The following terms are used in CodePipeline:
+ [Pipelines](#concepts-pipelines)
  + [Stages](#concepts-stages)
  + [Actions](#concepts-actions)
+ [Pipeline Executions](#concepts-executions)
  + [Stopped Executions](#concepts-executions-stopped)
  + [Failed Executions](#concepts-failed)
  + [Superseded Executions](#concepts-superseded)
+ [Action Executions](#concepts-action-executions)
+ [Transitions](#concepts-transitions)
+ [Artifacts](#concepts-artifacts)
+ [Source Revisions](#concepts-source-revisions)

### Pipelines<a name="concepts-pipelines"></a>

A *pipeline* is a workflow construct that describes how software changes go through a release process\. Each pipeline is made up of a series of *stages*\.

#### Stages<a name="concepts-stages"></a>

A stage is a logical unit you can use to isolate an environment and to limit the number of concurrent changes in that environment\. Each stage contains actions that are performed on the application [artifacts]()\. Your source code is an example of an artifact\. A stage might be a build stage, where the source code is built and tests are run\. It can also be a deployment stage, where code is deployed to runtime environments\. Each stage is made up of a series of serial or parallel *actions*\.

#### Actions<a name="concepts-actions"></a>

An *action* is a set of operations performed on application code and configured so that the actions run in the pipeline at a specified point\. This can include things like a source action from a code change, an action for deploying the application to instances, and so on\. For example, a deployment stage might contain a deployment action that deploys code to a compute service like Amazon EC2 or AWS Lambda\.

Valid CodePipeline action types are `source`, `build`, `test`, `deploy`, `approval`, and `invoke`\. For a list of action providers, see [Valid Action Types and Providers in CodePipeline ](reference-pipeline-structure.md#actions-valid-providers)\.

### Pipeline Executions<a name="concepts-executions"></a>

An *execution* is a set of changes released by a pipeline\. Each pipeline execution is unique and has its own ID\. An execution corresponds to a set of changes, such as a merged commit or a manual release of the latest commit\. Two executions can release the same set of changes at different times\.

While a pipeline can process multiple executions at the same time, a pipeline stage processes only one execution at a time\. To do this, a stage is locked while it processes an execution\. Two pipeline executions can't occupy the same stage at the same time\.

Pipeline executions traverse pipeline stages in order\. Valid statuses for pipelines are `InProgress`, `Stopping`, `Stopped`, `Succeeded`, `Superseded`, and `Failed`\. An execution with a `Failed` or `Superseded` status does not continue through the pipeline and cannot be retried\.

#### Stopped Executions<a name="concepts-executions-stopped"></a>

The pipeline execution can be stopped manually so that the in\-progress pipeline execution does not continue through the pipeline\. If stopped manually, a pipeline execution shows a `Stopping` status until it is completely stopped\. Then it shows a `Stopped` status\. A `Stopped` pipeline execution can be retried\.

There are two ways to stop a pipeline execution:
+ **Stop and wait**
+ **Stop and abandon**

For information about use cases for stopping an execution and sequence details for these options, see [How Pipeline Executions Are Stopped](concepts-how-it-works.md#concepts-how-it-works-stopping)\.

#### Failed Executions<a name="concepts-failed"></a>

If an execution fails, it stops and does not completely traverse the pipeline\. Its status is `FAILED` status and the stage is unlocked\. A more recent execution can catch up and enter the unlocked stage and lock it\. You can retry a failed execution unless the failed execution has been superseded or is not retryable\.

#### Superseded Executions<a name="concepts-superseded"></a>

To deliver the latest set of changes through a pipeline, newer executions pass and replace less recent executions already running through the pipeline\. When this occurs, the older execution is superseded by the newer execution\. An execution can be superseded by a more recent execution at a certain point, which is the point between stages\.

If an execution is waiting to enter a locked stage, a more recent execution might catch up and supersede it\. The newer execution now waits for the stage to unlock, and the superseded execution stops with a `SUPERSEDED` status\. When a pipeline execution is superseded, the execution is stopped and does not completely traverse the pipeline\. You can no longer retry the superseded execution after it has been replaced at this stage\.

For more information about superseded executions and locked stages, see [How Executions Are Processed in a Pipeline ](concepts-how-it-works.md#concepts-how-it-works-executions)\.

### Action Executions<a name="concepts-action-executions"></a>

An *action execution* is the process of completing a configured action that operates on designated [artifacts]()\. These can be input artifacts, output artifacts, or both\. For example, a build action might run build commands on an input artifact, such as compiling application source code\. Action execution details include an action execution ID, the related pipeline execution source trigger, and the input and output artifacts for the action\. Valid statuses for actions are `InProgress`, `Succeeded`, and `Failed`\.

### Transitions<a name="concepts-transitions"></a>

A *transition* is the point where a pipeline execution moves to the next stage in the pipeline\. You can disable a stage's inbound transition to prevent executions from entering that stage, and then you can enable the transition to allow executions to continue\. When more than one execution arrives at a disabled transition, only the latest execution continues to the next stage when the transition is enabled\. This means that newer executions continue to supersede waiting executions while the transition is disabled, and then after the transition is enabled, the execution that continues is the superseding execution\.

![\[A pipeline contains stages, which contain actions, which are separated by transitions that can be disabled and enabled.\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/images/pipeline-elements-workflow.png)![\[A pipeline contains stages, which contain actions, which are separated by transitions that can be disabled and enabled.\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)![\[A pipeline contains stages, which contain actions, which are separated by transitions that can be disabled and enabled.\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)

### Artifacts<a name="concepts-artifacts"></a>

*Artifacts* refers to the collection of data, such as application source code, built applications, dependencies, definitions files, templates, and so on, that is worked on by pipeline actions\. Artifacts are produced by some actions and consumed by others\. In a pipeline, artifacts can be the set of files worked on by an action \(*input artifacts*\) or the updated output of a completed action \(*output artifacts*\)\.

Actions pass output to another action for further processing using the pipeline artifact bucket\. CodePipeline copies artifacts to the artifact store, where the action picks them up\. For more information about artifacts, see [Input and Output Artifacts](welcome-introducing-artifacts.md)\.

### Source Revisions<a name="concepts-source-revisions"></a>

When you make a source code change, a new version is created\. A *source revision* is the version of a source change that triggers a pipeline execution\. An execution processes that source revision only\. For GitHub and CodeCommit repositories, this is the commit\. For S3 buckets or actions, this is the object version\.
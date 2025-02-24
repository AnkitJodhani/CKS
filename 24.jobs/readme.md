# Jobs
- The Job represents the overall task
- It run to completion and then stop
- Creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate

## Types of Jobs
1) NonIndexed - each Pod completion is homologous to each othe
2) Indexed  - the Pods of a Job get an associated completion index

## Attributes

- `completions: 5` # Number of successful Pods to complete jobs or task
- `parallelism: 2` # Number of pods at a time, default 1, and if set to "0" then the Job is effectively paused until it is increased
- `backoffLimit: 10` # Limit of retries before marking job as Failed

- `completionMode: Indexed` # or NonIndexed
- `backoffLimitPerIndex: 2`  # maximal number of failures per index
- `maxFailedIndexes: 5`      # maximal number of failed indexes before terminating the Job execution

## Backoff limit per index
```yml
  completionMode: Indexed  # required for the feature
  backoffLimitPerIndex: 1  # maximal number of failures per index
  maxFailedIndexes: 5      # maximal number of failed indexes before terminating the Job execution
```

## Pod failure policy
- if you want to use a .spec.podFailurePolicy field for a Job, you must also define
that Job's pod template with .spec.restartPolicy set to Never.

```yml
  template:
    spec:
      restartPolicy: Never #
  podFailurePolicy:
    rules:
    - action: FailJob
      onExitCodes:
        containerName: main      # optional
        operator: In             # one of: In, NotIn
        values: [42]
    - action: Ignore             # one of: Ignore, FailJob, Count
      onPodConditions:
      - type: DisruptionTarget   # indicates Pod disruption

```

## Success policy
- this policy can handle Job success based on the succeeded pods. After the Job meets the success policy, the job controller terminates the lingering Pods.

```yml
  completionMode: Indexed # Required for the success policy
  successPolicy:
    rules:
      - succeededIndexes: 0,2-3
        succeededCount: 1
```
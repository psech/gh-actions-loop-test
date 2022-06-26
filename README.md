# GitHub Actions loop test
The purpose of this project is to pass a sequence argument to a reusable workflow and run multiple builds.

## Approaches

1. A Simple workflow with a direct strategy
1. A reusable workflow with configuration JSON file
1. A reusable workflow accepting stringified JSON and input

### A Simple workflow with a direct strategy
Uses hardcoded matrix strategy to execute multiple jobs in parallel.

```
strategy:
  matrix:
    arr:
      - 3.1-sdk
      - 5.0-sdk
      - 6.0-sdk
```

### A reusable workflow with configuration JSON file
Since
> The `strategy` property is not supported in any job that calls a reusable workflow.

(see [Limitations](https://docs.github.com/en/actions/using-workflows/reusing-workflows#limitations))


The `file-based-workflow` uses a JSON file to build a strategy.

### A reusable workflow accepting stringified JSON and input

Since
> The `strategy` property is not supported in any job that calls a reusable workflow.

(see [Limitations](https://docs.github.com/en/actions/using-workflows/reusing-workflows#limitations))

and

The `on.workflow_call.inputs.<input_id>.type` accepts only 
- string
- number
- boolean

(see [docs](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_callinputsinput_idtype)), so a sequence (array) can't be provided.

The `from-json-workflow` uses stringified JSON and the input argument, that is later transvormed to a matrix strategy using `fromJSON` (see [fromJSON](https://docs.github.com/en/actions/learn-github-actions/expressions#fromjson)).
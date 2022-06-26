# GitHub Actions loop test
This project aims to pass a sequence argument to a reusable workflow and run multiple builds.

## Approaches

1. A Simple workflow with a direct strategy
1. A reusable workflow with a configuration JSON file
1. A reusable workflow accepting stringified JSON and input

## A Simple workflow with a direct strategy
The `loop-simple-workflow` uses a hardcoded matrix strategy to execute multiple jobs in parallel.

```
strategy:
  matrix:
    arr:
      - 3.1-sdk
      - 5.0-sdk
      - 6.0-sdk
```

## Challenges

1. The `strategy` property is not supported in any job that calls a reusable workflow. See [limitations](https://docs.github.com/en/actions/using-workflows/reusing-workflows#limitations)
1. The `on.workflow_call.inputs.<input_id>.type` accepts only `string`, `number` and `boolean`. It does not accept a sequence (array). See [docs](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_callinputsinput_idtype)

### Legend

1. File solution
    1. `loop-action-json-file` parent workflow
    1. `reusable-workflow-json-file` reusable workflow; disabled on purpose
1. Sequence solution
    1. `loop-action-sequence-pass` parent workflow
    1. `reusable-workflow-sequence-pass` reusable workflow; disabled on purpose

### Solution 1: A reusable workflow with a configuration JSON file

The file-based workflow uses a JSON file to build a strategy.

The parent workflow (`loop-action-json-file`) passes a config path to the reusable workflow.

```
uses: ./.github/workflows/reusable-workflow-json-file.yml
with:
  tags: ./settings.json
```

The reusable workflow (`reusable-workflow-json-file`):

1. sets outputs (same as solution 2)
    ```
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    ```
1. parses a file and sets output variable
    ```
    run: |
      echo Setting matrix output from file
      JSON=$(cat ${{inputs.tags}})
      echo "::set-output name=matrix::${JSON//'%'/'%25'}"
    ```
1. sets the strategy (same as solution 2)
    ```
    strategy:
      matrix:
        tags: ${{fromJSON(needs.set-strategy.outputs.matrix)}}
    ```
1. run parallel builds

### Solution 2: A reusable workflow accepting stringified JSON as an input

The parent workflow (`loop-action-sequence-pass`) passes hardcoded JSON to the reusable workflow.

1. sets the output (same as solution 1)
    ```
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    ```
1. sets the matrix
    ```
    run: |
      echo "::set-output name=matrix::['3.1-sdk', '5.0-sdk', '6.0-sdk']"
    ```
    Note: There should be a way to provide a YML config and use `toJSON()`.
1. passes JSON string to the reusable workflow
    ```
    uses: ./.github/workflows/reusable-workflow-sequence-pass.yml
    with:
      tags: ${{needs.tags-matrix.outputs.matrix}}
    ```

The reusable workflow (`reusable-workflow-sequence-pass`)

1. sets outputs
    ```
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    ```
1. sets output variable
    ```
    run: |
      echo Setting matrix output from parent workflow
      echo "::set-output name=matrix::${{inputs.tags}}"
    ```
1. sets the strategy (same as solution 1)
    ```
    strategy:
      matrix:
        tags: ${{fromJSON(needs.set-strategy.outputs.matrix)}}
    ```
1. run parallel builds
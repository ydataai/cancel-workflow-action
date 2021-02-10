# This fork:

This fork helps to cancel the runs in all branchs, when you use the workflow with `on: [prereleased released]`.

Without this changes, the cancel-worflow will check the runs running in the specific released or prereleased tag. 

For example, if you have a prerelease, with a tag "1.1.1" , the cancel-workflow will check the runs running with the `head_branch` (from the API) = 1.1.1 and ignore all the runs that start with the previous released or prereleased tags.

Ignoring the branch, the cancel-workflow will call the API and get all the runs that match the `workflow_id` and cancel this runs.

Usage example:

```yml
name: Prereleased

on:
  release:
    types: [ prereleased ]

jobs:
  cancel_previous:
    name: 'Cancel Previous Runs'
    runs-on: ubuntu-20.04
    timeout-minutes: 3
    steps:
      - uses: ydataai/cancel-workflow-action@0.7.0-fix
        with:
          access_token: ${{ github.token }}
```

# Cancel Workflow Action

This is a GitHub Action that will cancel any previous runs that are not `completed` for a given workflow.

This includes runs with a [status](https://docs.github.com/en/rest/reference/checks#check-runs) of `queued` or `in_progress`.

## How does it work?

When you `git push`, this GitHub Action will capture the current Branch and SHA. It will query GitHub's API to find previous workflow runs that match the Branch but do not match the SHA. These in-progress runs will be canceled leaving only the latest run.

Read more about the [Workflow Runs API](https://docs.github.com/en/rest/reference/actions#workflow-runs).

## Usage

Typically, you will want to add this action as the first step in a workflow so it can cancel itself on the next push.

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Cancel Previous Runs
        uses: styfle/cancel-workflow-action@0.7.0
        with:
          access_token: ${{ github.token }}
      #- name: Run Tests
      #  uses: actions/setup-node@v1
      #  run: node test.js
      # ... etc
```


### Advanced

In some cases, you may wish to avoid modifying all your workflows and instead create a new workflow that cancels your other workflows. This can be useful when you have a problem with workflows getting queued.

- Visit `https://api.github.com/repos/:org/:repo/actions/workflows` to find the Workflow ID(s) you wish to automaticaly cancel.
- Add a new file `.github/workflows/cancel.yml` with the following:

```yml
name: Cancel
on: [push]
jobs:
  cancel:
    name: 'Cancel Previous Runs'
    runs-on: ubuntu-latest
    timeout-minutes: 3
    steps:
      - uses: styfle/cancel-workflow-action@0.7.0
        with:
          workflow_id: 479426
          access_token: ${{ github.token }}
```

- _Note_: `workflow_id` can be a Workflow ID (number) or Workflow File Name (string)
- _Note_: `workflow_id` also accepts a comma separated list if you need to cancel multiple workflows

In some cases, you may wish to cancel workflows when you close a Pull Request. Because this is not a push event, the SHA will be the same, so you must use the `ignore_sha` option.

```yml
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    name: 'Cleanup After PR Closed'
    runs-on: ubuntu-latest
    timeout-minutes: 3
    steps:
      - name: Cancel build runs
        uses: styfle/cancel-workflow-action@0.7.0
        with:
          ignore_sha: true
          workflow_id: 479426
```

## Contributing

- Clone this repo
- Run `yarn install`
- Edit `./src/index.ts`
- Run `yarn build`
- Commit changes including `./dist/index.js` bundle

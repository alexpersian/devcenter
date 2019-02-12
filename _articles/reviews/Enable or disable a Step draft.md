---
title: Enabling or disabling a Step conditionally - draft
redirect_from: []
date: 2019-02-12 11:15:35 +0000
published: false

---
You can enable or disable a Step in any given workflow, and you can also set conditions for Steps. You can do it either on your own machine, with the Bitrise CLI or by using the `bitrise.yml` tab of the Workflow Editor.

## Disabling a Step

If you do not want to remove a Step from your workflow but you don't want it to run, you can disable it, using a `run_if` expression.

1. Open your app's `bitrise.yml` file.
2. Find the Step that you want to disable.
3. Add `run_if: false` to it.

Example:

    - script:
        run_if: false
        inputs:
        - content: |-
            #!/bin/bash
            echo "This will never run, because of run_if:false"

{% include message_box.html type="note" title="Experimenting with workflows" content="To experiment with different configurations for a workflow, without removing or disabling Steps, we recommend cloning the workflow. You can modify the cloned workflow as much as you wish without changing anything in the original."%}

## Running a Step only in CI environment

Running a Step only in a CI environment means your build will skip that particular Step for local builds. Like disabling Steps, you can do this with a `run_if` expression. Use this to debug builds locally.

1. Open your app's `bitrise.yml` file.
2. Find the Step that you want to disable.
3. Add `run_if: .IsCI` to it.

Example:

    - script:
        run_if: .IsCI
        inputs:
        - content: |-
            #!/bin/bash
            echo "This will only ever run in a CI environment because run_if: IsCI"

{% include message_box.html type="info" title="My message" content="Many Steps have this .`IsCI` flag set by default: for example, the `Git Clone` Step. However, you can change the `run_if` property of these Steps, too: just set it to `run_if: true`."%} 

{% include message_box.html type="info" title="Enable CI mode" content=" CI mode can be enabled on your own Mac/PC by setting the `CI` environment to `true` (for example, run `export CI=true` in your Bash Terminal), or by running `bitrise run` with the `--ci` flag: `bitrise --ci run ...`. "%}

## Run a Step only if the build failed

_To do this you have to switch to_ `_bitrise.yml_` _mode (open the Workflow Editor on bitrise.io -> left side: click on_ `_bitrise.yml_` _to switch to the interactive_ `_bitrise.yml_` _editor)._

You have to add two properties to the Step you **only** want to run when the Build failed (at that point, when the Step would run):

* `is_always_run: true` (this enables the Step to be considered to run even if a previous Step failed)
* `run_if: .IsBuildFailed` (you can find more examples of the `run_if` template at: [https://github.com/bitrise-io/bitrise/blob/master/_examples/experimentals/templates/bitrise.yml](https://github.com/bitrise-io/bitrise/blob/master/_examples/experimentals/templates/bitrise.yml "https://github.com/bitrise-io/bitrise/blob/master/_examples/experimentals/templates/bitrise.yml")).

An example `script` step, which will only run if the Build failed:

    - script:
        is_always_run: true
        run_if: .IsBuildFailed
        inputs:
        - content: |-
            #!/bin/bash
            echo "Build Failed!"

{% include message_box.html type="note" title="A **run_if** can be any valid **Go** template" content=" A `run_if` can be any valid [Go template](https://golang.org/pkg/text/template/), as long as it evaluates to `true` or `false` (or any of the String representation, e.g. `\"True\"`, `\"t\"`, `\"yes\"` or `\"y\"` are all considered to be `true`). If the template evaluates to `true` the Step will run, otherwise it won't. "%}

An example `run_if` to check a **custom environment variable** (you can expose environment variables from your scripts too, using [envman](https://github.com/bitrise-io/envman/)):

    {% raw %}
    run_if: |-
     	{{enveq "CUSTOM_ENV_VAR_KEY" "test value to test against"}}
    {% endraw %}    

This `run_if` will skip the step in every case when the value of `CUSTOM_ENV_VAR_KEY` is not `test value to test against`.
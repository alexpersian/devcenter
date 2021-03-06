---
title: 'Triggering builds - draft '
redirect_from: []
date: 2019-01-16 15:45:49 +0000
published: false

---
You can trigger and abort builds with the Bitrise API. You can define parameters for the build like what `branch`, `tag` or _git commit_ to use and what _build message_ to present on the Build's details page.

## Triggering a new build

To trigger a new build with the Bitrise API, call a `POST` request with a JSON body. Here's an example `curl` request:

``` bash
curl -X POST -H 'Authorization: token THE-ACCESS-TOKEN' 'https://api.bitrise.io/v0.1/apps/APP-SLUG/builds' -d '{"hook_info":{"type":"bitrise"},"build_params":{"branch":"master","workflow_id":"primary"},"triggered_by":"bitrise_api_doc"}'
```

{% include message_box.html type="note" title="Interactive cURL call configurator" content="You can find an interactive cURL call configurator by clicking on the `Start/Schedule a build` button on your app's [bitrise.io](https://www.bitrise.io) page and switching to `Advanced` mode in the popup. At the bottom of the popup you can find a `curl` call, based on the parameters you specify in the popup. "%}

Let's break this request down!

### The JSON body

The JSON body has to contain at least:

* a `hook_info` object with:
  * a `type` key and `bitrise` as its value
* a `build_params` object, with at least one of the following parameters specified:
  * `tag`
  * `branch`
  * `workflow_id`

Here's a minimal sample JSON body which specifies _master_ as the value of the `branch` parameter:

    {
      "hook_info": {
        "type": "bitrise",
      },
      "build_params": {
        "branch": "master"
      }
    }

In our earlier example, we passed this JSON payload as a string: to be precise, as a JSON object serialized to a string.

You can also pass it as an object (for example, if you want to call it from JavaScript. To do so, include a root `payload` element or, alternatively, set the JSON object as the value of the `payload` POST parameter.

Here's a jQuery example using the `payload` parameter:

    $.post("https://api.bitrise.io/app/APP-SLUG/builds/", {
        "payload":{
            "hook_info":{
                "type":"bitrise",
            },
            "build_params":{
                "branch":"master"
            }
        }
    })

## Build Parameters

The following parameters are supported in the `build_params` object:

### Git related params:

* `branch` (string): The (Source) Branch to build. In the case of a standard git commit this is the branch of the commit. In the case of a Pull Request build this is the source branch, the one the PR was started from.
* `tag` (string): The git Tag to build.
* `commit_hash` (string): The git commit hash to build.
* `commit_message` (string): The git commit message (or build's message).

### Bitrise-specific params:

* `workflow_id`: (string): Force the use of the specified workflow ID. If not defined then the workflow will be selected based on the project's [Trigger Map config](/webhooks/trigger-map/).
* `environments` (array of objects): See the [Specify Environment Variables](#specify-environment-variables) section for more info about the `environments` objects.
* `skip_git_status_report` (bool): Skip sending build status for the connected git provider

### Pull Request specific params:

* `branch_dest` (string): Used only for Pull Request builds: the destination/target branch of the Pull Request, the one the PR will be merged _into_.
  Example: `master`.
* `pull_request_id` (int): Pull Request ID on the source code hosting system (for example, the PR number on GitHub).
* `pull_request_repository_url` (string): repository url from where the Pull Request is sent. For example, if the PR is created from a fork this should be the fork's URL.
  Example: `https://github.com/xyz/bitrise.git`.
* `pull_request_merge_branch` (string): the pre-merge branch, **if the source code hosting system supports & provides** the pre-merged state of the PR on a special "merge branch" (ref).
  Example: `pull/12/merge`.
* `pull_request_head_branch` (string): the Pull Request's "head branch" (`refs/`). This special git `ref` should point to the **source** of the Pull Request. Supported by GitHub and GitLab.
  Example: `pull/12/head` (github) / `merge-requests/12/head` (gitlab).

{% include message_box.html type="note" title="Git Clone - parameter priority" content=" If you provide a `tag`, the `branch` parameter will be ignored by the `Git Clone` step. If you provide a `commit_hash` parameter then both the `tag` and the `branch` parameters will be ignored. These will still be logged, will be available for steps and will be visible on the Build's details page, but the `Git Clone` tep will use the the most specific parameter for checkout. "%}

### Specify Environment Variables

You can define additional _environment variables_ for your build.

_These variables will be handled with priority between_ `Secrets` _and_ `App Env Vars`_, which means that you can not overwrite environment variables defined in your build configuration (e.g. App Env Vars), only Secrets. For more information see: _[Availability order of environment variables](/bitrise-cli/most-important-concepts/#availability-order-of-environment-variables)

It's important that this parameter have to be an **array of objects**, and that every item of the array have to include at least a `mapped_to` (the key of the Environment Variable, without a dollar sign (`$`)) and a `value` property (the value of the variable). By default environment variable names inside values will be replaced in triggered build by actual value from target environment. This behavior can be disabled by setting `is_expand` flag to `false`.

Example:

    "environments":[
      {"mapped_to":"API_TEST_ENV","value":"This is the test value","is_expand":true},
      {"mapped_to":"HELP_ENV","value":"$HOME variable contains user's home directory path","is_expand":false},
    ]

### Workflow to be used for the build

By default the Workflow for your Build will be selected based on the `build_params` and your app's [Trigger Map](/webhooks/trigger-map/). This is the same as how [Webhooks](/webhooks/) select the workflow for the build automatically (based on the _Trigger Map_), and how you can define separate Workflows for separate branches, tags or pull requests without the need to specify the workflow manually for every build.

With the Trigger API you can however **overwrite** this selection and specify exactly which Workflow you want to use.

All you have to do is add a `workflow_id` parameter to your `build_params` and specify the Workflow you want to use for that specific build.

An example `build_params` with `branch` and `workflow_id`:

    "build_params":{"branch":"master","workflow_id":"deploy"}'

## `curl` example generator

You can find an interactive cURL call configurator by clicking on the `Start/Schedule a build` button on your app's [bitrise.io](https://www.bitrise.io) page and switching to `Advanced` mode in the popup. At the bottom of the popup you can find a `curl` call, based on the parameters you specify in the popup.

A base curl call would look like this (with `master` specified as the `branch` build parameter):

    curl -H 'Content-Type: application/json' https://app.bitrise.io/app/APP-SLUG/build/start.json --data '{"hook_info":{"type":"bitrise","build_trigger_token":"APP-API-TOKEN"},"build_params":{"branch":"master"}}'

_Note: please don't forget to add_ `Content-Type` _header with_ `application/json` _value_

A more advanced example: let's say you want to build the **master** `branch` using the `deployment` workflow, specify a build message (`commit_message`) and set a test environment variable (`API_TEST_ENV`), the call will look like this:

    curl  -H 'Content-Type: application/json' https://app.bitrise.io/app/APP-SLUG/build/start.json --data '{"hook_info":{"type":"bitrise","build_trigger_token":"APP-API-TOKEN"},"build_params":{"branch":"master","commit_message":"Environment in API params test","workflow_id":"deployment","environments":[{"mapped_to":"API_TEST_ENV","value":"This is the test value","is_expand":true}]}}'

## Aborting a build
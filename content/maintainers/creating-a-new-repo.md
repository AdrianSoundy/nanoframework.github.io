# Procedure for creating a new repository

**About this document**

This document describes the recipe to create a new GitHub repository. It's meant for class libraries.

## Introduction

The strict following of this procedure is required in order to maintain consistency and coherence throughout the repositories, along with taking advantage of the build tools, testing and publishing automation.
If in doubt please ask one of the senior team members.

## Creating the repository in GitHub

1. This is basically clicking the create new repository button in GitHub. 

    Note: The class libraries repositories are following the patter "<strong>lib-</strong><i>namespace</i>" most of the remaining repositories "<strong>nf-</strong><i>some-relevant-name-here</i>". This makes it easier to spot what is what.

2. As we are following the [GitFlow branching model](http://nvie.com/posts/a-successful-git-branching-model/) two branches must be created: `master` and `develop`.

3. Make sure to create an empty readme.md to make it easier to fork and clone the new repo.

## Adjust the repository settings (part 1)

1. Go to the repository **Settings** and move into _Options_.

2. In the _Features_ section disable Wikis, Issues and Projects.

3. On the _Merge Button_ section disable Allow merge commits. We prefer to have tidy merges on PRs without having to bother contributors to squash commits.

4. Move into _Branches_ and set `develop` as the default branch.

## Setup the CLA

1. Open a browser window in Private Mode (so you can sign-in as `nfbot` and not loose you personal GitHub session).

2. Navigate to the [CLA Assistant](https://cla-assistant.io/).

3. Sign-in with GitHub `nfbot` account.

4. Click the "Configure CLA" button at the top left.

5. Select the newly created repo on the "Choose a repository" drop-down.

6. Select the "nanoFramework-CLA.md" in the next drop-down.

7. Click the "Link" button and agree with the next step.

8. On the list of the CLAs, find the new one and click on the ellipsis to the right and then 'Edit'.

9. Add `nfbot,*[bot]` into the field "Provide user names, who doesn't need to sign the CLA". Click _Save_.

## Setup Azure DevOps

1. Open a new browser window on which you are signed in to GitHub as `nfbot`.
1. On the repo Settings, navigate to "Integrations & Services".
1. Click "Configure" button for Azure Pipelines.
1. Scroll down to "Repository Access" and click "Save".
    - If the Save button is not enabled. Switch the radio buttons with the options until it does. Make sure that you leave that on "All repositories" option.
1. The next step will take you to the [Azure DevOps](https://dev.azure.com/nanoframework) website.
1. Click on "Create New Project".
1. Name the project following the GiHub repo name but without the "lib" prefix. Make it _Public_, select _Git_ as the version control and _Agile_ as the work item process.
1. After the project is created and showing the Summary page, go to the _Project Settings_ (cog wheel at the bottom left) and navigate to _Service Connections_.
1. Click "New service connection" and choose _NuGet_.
1. Enter the details for a connection named `AzurePipelines`, with URL `https://pkgs.dev.azure.com/nanoframework/feed/_packaging/sandbox/nuget/v3/index.json` and PAT token for nanoFramework Azure DevOps account.
1. Repeat the previous step and enter the details for another connection named `NuGet`, with URL `https://api.nuget.org/v3/index.json` and API key from nanoFramework NuGet API.
1. Click again on "New service connection" and choose _SonarCloud_.
1. Enter the details for a connection named `sonarcloud`, with the token for nanoFramework SonarCloud account.
1. Navigate to Pipelines and click "New pipeline".
1. Choose _GitHub_ on the option "Where is my code?" and authorize with OAuth.
1. On the next step select the new repository from the drop down list.
1. As a template choose "Starter pipeline" to get things moving and then click on "Save and run". This happens twice to make the commit and start the very first build.
1. Click the ellipsis icon on the top right and go to "Edit pipeline".
1. After the pipeline view loads, choose _Hosted VS2017_ in the Agent pool list.
1. Move to "Get sources" and verify that the default branch is `develop`.
1. Navigate to "Variables" and add `DiscordWebhook` with a value taken from the Azure webhook of the "build-monitor" channel in our Discord server. **Make sure** that the variable is set to `secret` by clicking on the padlock icon.
1. Add another variable `GitHubToken` with a value taken from the nfbot personal tokens in GitHub. **Make sure** that the variable is set to `secret` by clicking on the padlock icon.
1. Navigate to "Triggers". Make sure that the option to override YAML is **not** checked for "Continuous integration". Uncheck the same option for "Pull request validation" and check the "Make secrets available to builds of forks".
1. Click the "Save" button and confirm the operation on the pop-up.
1. Go back to the pipelines view and with the current pipeline selected, click on the ellipsis icon and then on "Status badge". Copy the markdown code that shows on the pop-up. This will be required to add the correct build badges in the repo readme in a moment.

## Prepare the initial commit

1. Fork the repo into your preferred GutHub account and clone it locally.
1. The best option is to copy/paste from an existing repo, so you're more efficient doing just that. Mind the name changes tough! Grab the following files:
    - .github\PULL_REQUEST_TEMPLATE.md _(no changes required)_
    - .gitignore _(no changes required)_
    - .github_changelog_generator
    - azure-pipelines.yml
    - CODE_OF_CONDUCT.md _(no changes required)_
    - CONTRIBUTING.md _(no changes required)_
    - LICENSE _(no changes required)_
    - README.md
    - template.vssettings _(no changes required)_
    - source/version.json
    - source/NuGet.Config
    - source/readme.txt
    - update-dependencies.ps1 _(this is only required if this class library is a dependency of others)_
1. Open "azure-pipelines.yml"
    1. Rename the `nugetPackageName` variable with the new name (mind the nanoframework prefix).
    1. Rename the `repoName` variable with the repo name.
    1. Rename the `sourceFileName` parameter with the equivalent name. It's probably wise to wait for the first successful build of the class library and then get back here with the correct name for the assembly declaration source file.
    1. Rename the `sonarCloudProject` variable with the repo name.
    1. If there are class libraries that depend on this one, copy the "update dependencies" step from CorLib "azure-pipelines.yml". If there aren't just skip this step.
1. Open ".github_changelog_generator" and set the _project_ to the repo name.
1. Open "source\version.json" and set the _version_ to the appropriate one. Make sure to follow our version number guidelines. In doubt please ask one of the senior team members.
1. Open "README.md"
    1. Rename the class library name occurrences with the new name.
    2. Rename the package name for the NuGet badges.
    3. Replace the build status badges with the ones that you've copied from Azure DevOps. They'll be the same until there is a second pipeline for the master branch.
1. Create a "source" folder that will hold the code files and VS Solutions and projects.
1. Create a folder inside "source" with the name of the new class library.
1. Add to the VS Solution the class library project. Again it's better to follow an existing one and ask in doubt.
    1. Make sure you are following the naming pattern.
    1. Do not add a `.nuget` solution folder
    1. Make sure you copy the `key.snk` from the initial repo (or from the CorLib repo). **DO NOT** create a new one.
1. Rename, edit and adjust as required the "nuspec" files to create the NuGet packages.
1. Edit the "readme.txt" inside the source folder and rename the repository name.
1. Still on"azure-pipelines.yml" _and only_ if there are class libraries that depend on this one.
    1. Adjust the `repositoriesToUpdate` list with the repo names of the class libraries that depend on this new one.

## Adjust the repository settings (part 2)

1. Go to the repository settings in GitHub and move into _Branches_.
1. Go to the rule for "develop" branch and change the following:
      - Enable "Require pull request reviews before merging"
      - Enable "Require status checks to pass before merging" with the options:
        - "Require branches to be up to date before merging"
        - "Status checks: nanoframework.<i><strong>azure-devops-project-name</strong></i>"
        - "Status checks: license/cla" (for develop branch)

## Update the dependency upwards

As a minimum, the new class library depends on mscorlib. If that's the only dependency, edit the [`azure-pipelines.yml`](https://github.com/nanoframework/lib-CoreLibrary/blob/develop/azure-pipelines.yml) file there and add this new repo to the `repositoriesToUpdate` list.
Now, if it depends on others, you have to figure out which one of those is _at the end_ of the dependency chain and add this new repo to _that_ `azure-pipelines.yml` file. For example, `Windows.Devices.Gpio` depends on `CoreLibrary` and `Runtime.Events` (which, in turn, depends on `CoreLibrary`). Updating it's dependencies has to the triggered at `Runtime.Events` not on `CoreLibrary` because of the chained dependency.

## Add the class library to the documentation project

If this class library has documentation that has to be published as part of nanoFramework documentation (which is most likely) it needs to be referenced in the documentation project.
Edit the documentation repo [`azure-pipelines.yml`](https://github.com/nanoframework/nanoframework.github.io/blob/pages-source/azure-pipelines.yml) and add entries for this new repo on the clone, restore and build steps.

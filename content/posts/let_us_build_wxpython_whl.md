---
title: Let us build wxPython whl
description: Speed up the build and release process.
tags:
 - wxPython
 - AWSManager
 - Workdir
 - GitHubIssueClient
 - RunIfExists
 - VATValidation
date: 2024-07-21
---

It doesn't matter, which application I build. The build step of pyinstaller runs about half an our only because of compiling the wxPython whl file. This whl package is used to create the binary file for Linux.

There are some ready to use whl files as [wxPython extras](https://extras.wxpython.org/wxPython4/extras/linux/ "xPython extras"). But they do not fit for my environment. Therefore I have to find a way, how can I build it on my own and reuse it for all my application.

## First step

I create my own repository [dseichter/wxpython-whl](https://github.com/dseichter/wxpython-whl "wxPython whl") and added a simple workflow. 

## The workflow

The workflow will just do the following:

* Get the version I added to the requirements.txt file
* Trigger the workflow to build the whl file
* Create a release, that provides the whl file

The first positive side effect: While I am using the same docker image within the runner, I can be sure, this file is compatible. And the second positive side effect: Using dependabot will keep me informed, if there are updates -> perfect.

So I started to read about how to build a wxPython wheel file, using this [blog post](https://wxpython.org/blog/2017-08-17-builds-for-linux-with-pip/index.html "wxPython builds for linux").

With my experience in writing GitHub Action workflows, it was more or less easy to get the correct information into the right order to build my own wheel file for wxPython. 

The location of my workflow is this: [workflow file](https://github.com/dseichter/wxpython-whl/blob/main/.github/workflows/build-whl.yml "complete workflow file")

I only want to talk more about two special topics within my workflow file. The most important thing is, to be dynamic as much as possible. Therefore I added this step:

```YAML
- name: Get version of wxPython out of requirements.txt
    id: get_wx_version
    run: |
    echo "wx_version=$(cat requirements.txt | grep wxPython | cut -d'=' -f3)" >> $GITHUB_OUTPUT
```

With this step, I am able to get the version out of the requirements file. This I can use in all the later steps with ```${{ steps.get_wx_version.outputs.wx_version }}```.

Second thing I want to mention is the available GitHub Action [softprops/action-gh-release](https://github.com/softprops/action-gh-release/tree/v2/ "softprops/action-gh-release"). This action helps a lot on automate the release process and persist the build artifacts as releases.

```YAML
- name: Create Release
    id: create_release
    uses: softprops/action-gh-release@v2
    with:
    tag_name: ${{ steps.get_wx_version.outputs.wx_version }}
    name: wxPython WHL ${{ steps.get_wx_version.outputs.wx_version }}
    draft: false
    prerelease: false
    generate_release_notes: true
    files: |
        wheel-file
```

You see the usage of the output. If I will rerun the pipeline again, it will update this release.

That's it :smile: Hope you enjoy reading and you can get some useful information.
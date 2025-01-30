---
title: "Running GitHub Actions on a randomized schedule"
date: "2023-03-30T20:40:00.000-04:00"
lastmod: "2025-01-30T05:03:00.000Z"
draft: false
series: []
authors:
  - "Peter"
tags:
  - "engineering"
categories:
  - "eng"
NOTION_METADATA:
  object: "page"
  id: "16ac6bf0-7757-80ae-a0d9-ff5686d09f1a"
  created_time: "2024-12-28T15:48:00.000Z"
  last_edited_time: "2025-01-30T05:03:00.000Z"
  created_by:
    object: "user"
    id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
  last_edited_by:
    object: "user"
    id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
  cover: null
  icon: null
  parent:
    type: "database_id"
    database_id: "167c6bf0-7757-810a-95f9-cfb3b2adb3e1"
  archived: false
  in_trash: false
  properties:
    series:
      id: "B%3C%3FS"
      type: "multi_select"
      multi_select: []
    draft:
      id: "JiWU"
      type: "checkbox"
      checkbox: false
    published-date:
      id: "%5C%5CDZ"
      type: "date"
      date:
        start: "2023-03-30T20:40:00.000-04:00"
        end: null
        time_zone: null
    authors:
      id: "bK%3B%5B"
      type: "people"
      people:
        - object: "user"
          id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
          name: "Peter"
          avatar_url: "https://s3-us-west-2.amazonaws.com/public.notion-static.com/c7114a\
            e2-5f13-4f26-95a6-9feccbcca818/profile-pic_(1).png"
          type: "person"
          person:
            email: "notion@peterrauscher.com"
    custom-front-matter:
      id: "c~kA"
      type: "rich_text"
      rich_text: []
    tags:
      id: "jw%7CC"
      type: "multi_select"
      multi_select:
        - id: "d3648cb9-022a-42c7-a0bc-6601edae5057"
          name: "engineering"
          color: "green"
    categories:
      id: "nbY%3F"
      type: "multi_select"
      multi_select:
        - id: "6cfa2b44-9f5c-40f5-8f84-24c6a8151155"
          name: "eng"
          color: "purple"
    summary:
      id: "x%3AlD"
      type: "rich_text"
      rich_text: []
    Name:
      id: "title"
      type: "title"
      title:
        - type: "text"
          text:
            content: "Running GitHub Actions on a randomized schedule"
            link: null
          annotations:
            bold: false
            italic: false
            strikethrough: false
            underline: false
            code: false
            color: "default"
          plain_text: "Running GitHub Actions on a randomized schedule"
          href: null
  url: "https://www.notion.so/Running-GitHub-Actions-on-a-randomized-schedule-16a\
    c6bf0775780aea0d9ff5686d09f1a"
  public_url: "https://peterrauscher.notion.site/Running-GitHub-Actions-on-a-rand\
    omized-schedule-16ac6bf0775780aea0d9ff5686d09f1a"
MANAGED_BY_NOTION_HUGO: true

---


GitHub Actions provide a powerful automation platform that allows developers to build, test, and deploy their projects with ease. By default, workflows are scheduled at fixed times using the cron syntax. However, sometimes it can be beneficial to run actions at random intervals to avoid predictable patterns or distribute resource usage. Let’s explore a method I whipped up to run GitHub Actions on a randomized schedule by randomizing crontab string with the `date` and `sed` commands.


## Setting up the Randomized Workflow


To implement a randomized GitHub Actions workflow, we will create a bash script named `randomize-workflow.sh`. This script will generate a random minute value and update the workflow's cron schedule accordingly.


```bash
#!/bin/bash

workflow_file=".github/workflows/example.yml"
minute=$(shuf -i 0-58 -n 1)
cron_string="$minute * * * *"

# Use sed to find and replace the cron line in the workflow file
sed -i "s/^    - cron:.*/    - cron: $cron_string/" "$workflow_file"

```


The script first defines the location of the workflow file and then generates a random minute value between 0 and 58 using `shuf`. Next, it constructs a new cron string with the randomized minute value and uses `sed` to update the corresponding line in the workflow file.


In the example workflow file `example.yml`, we have a job named "download" that runs on a fixed schedule every hour (at the 1st minute of each hour). We will replace this fixed cron schedule with our randomized cron schedule using the `randomize-workflow.sh` script.


Before using this script, make sure to add a secret named `COMMIT_TOKEN` to your GitHub repository. The `COMMIT_TOKEN` should contain an access token with read/write permissions to the repository running the GitHub Actions workflow. This token will be used to commit and push the changes made by the script.


## Modifying the GitHub Workflow


The `example.yml` GitHub Actions workflow needs a few adjustments to integrate the randomization script and use the secret token. Here's the modified workflow:


```yaml
name: Example Random Workflow

on:
  push:
    branches: [main]
  workflow_dispatch:
  schedule:
    - cron: 1 * * * *

jobs:
  download:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
          token: ${{ secrets.COMMIT_TOKEN }}
      - name: Randomize time for next run
        run: |
          chmod +x ./randomize-workflow.sh
          ./randomize-workflow.sh
      - name: Commit & push changes
        run: |
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config user.name "github-actions[bot]"
          git add .github/workflows/example.yml
          git commit -m "Randomized GitHub Actions workflow [skip ci]"
          git pull --rebase
          git push
```


In the updated workflow, we added a new step named "Randomize time for next run." This step runs the `randomize-workflow.sh` script, which modifies the workflow's cron schedule.


## Conclusion


Randomizing GitHub Actions' cron schedule can be a useful approach to distribute resource usage and avoid predictable patterns (especially for scraping/rate limited IO operations). By using a simple bash script with `date`, `sed`, and the appropriate secret token, we can update the cron schedule of the workflow programmatically.


Remember that while randomizing the schedule can reduce resource contention, it may also introduce challenges in coordination and monitoring. It's essential to strike a balance between “randomization” and predictable workflow execution based on your project's requirements.


Now you have the knowledge to implement randomized GitHub Actions workflows to enhance the automation and scheduling capabilities of your projects. Happy automating!


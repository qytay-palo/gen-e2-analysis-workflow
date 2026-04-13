---
description: Generate User Stories from Problem Statements for Data Science and Analytics
stage: Project Management
subcategory: subcategory-agile-tools
rule_name: user-stories
rule_version: latest
---

# Tasks

You are a expert Agile Business Analyst preparing problem statement, user stories and implementation plans for data science and analytics projects. For each problem statements: 

## 1. Identify problem statement

run identify-problem-statement subagent to identify and prioritize actionable analytics problem statements based on the business context, data availability, and potential impact. Details can be found in [../agents/identify-problem-statement]. Where possible run parallel for multiple problem statements.

## 2. Generate User Stories from Problem Statements

run generate-user-stories subagent in parallel for all problem statements if there are multiple problem statements, details can be found in [../agents/generate-user-stories] to generate user stories for each user story. 

Validate and ensure that each user stories is comprehensive, covering all necessary steps, tools, and best practices for production-grade data analytics pipelines and is found in the respective folders under `docs/objectives/user_stories/problem-statement-{num}-{name}/<numerical_prefix>-<story_name>.md`. Each user story should represent a distinct piece of functionality or value from an end-user perspective, following best practices in user story design and the data analysis lifecycle.

## 3. Generate Implementation Plan for User Stories

Once user stories are generated, run generate-implementation-plan subagent for all user stories in parallel, details can be found in [../agents/generate-implementation-plan] to generate implementation plan for each user story.

Keep track and ensure that each implementation plan is comprehensive, covering all necessary steps, tools, and best practices for production-grade data analytics pipelines and is appended under each user story file.
---
title: "An AI Agent Is Not Just a Code Generator — It's an Autonomous Engineer. But Only If You Give It 'Eyes'."
date: 2026-02-23T16:56:40+01:00
tags : [
    "feedback loop",
    "dotnet",
    "ci/cd",
    "azure devops",
    "autodocumentation",
    "sql server"]
---

The real power of an AI agent comes when we let it verify its own work — when we give it a feedback loop.

<!--more-->

We work with dotnet, and the solution is straightforward.
I moved away from partial automation to building a full autonomous development loop:

- Created Dockerfiles to build microservice projects, with all linters configured (all warnings as errors), dotnet format, dotnet test, and more
- Created a docker-compose to run all services, including infrastructure (database, message broker, etc.)
- Added OpenTelemetry to the code and set up metrics and traces collection
- Added Jaeger for trace visualization and Prometheus for metrics collection to docker-compose
- Created smoke tests to verify core functionality, which run during image builds and container startup

Now the agent can work as an autonomous developer and run the full development cycle:
- modify the code
- run linters, static analysis, and other checks
- test regression for standard functionality and investigate specific issues
- read container logs, monitor metrics and traces to find problems if we have a regression, or verify how a new feature works

The prompt generally looks like this:
```
1. Plan changes for the new feature
...
2. Plan e2e tests to verify the new feature
3. Make changes to the code
4. Build a new Docker image
5. Restart the test environment
6. Run the tests
7. Verify the new feature through e2e tests, logs, metrics, and traces
```

This allows us to work in a single tool without distractions. The agent gets all the information it needs to verify its own work. It can:
- make direct queries to the database or the message broker via API, and more
- check application logs
- check metrics and traces
- run tests and review results
- and much more that can be automated

When using MS GitHub Copilot Chat, this counts as a single premium request instead of 10 separate ones for each step, plus there is a higher chance the agent will complete the entire feature in one request.

The development cycle becomes more effective and faster.

But what do you do when this approach is not available? For example, when you are writing a CI/CD pipeline.
The cycle looks like this: edit YAML files, push the code, wait for the pipeline to start, and check the result. If there is an error, you need to go to the pipeline, check the logs, copy them into the chat, and ask the agent to help find the problem. Then repeat. Instead of taking 10–20 minutes, the development can take 30–40 minutes or even longer. This is inefficient.

You need to break down the task and look for tools that let you set up the feedback loop within the agent's work session again.

Here is an example: I had a task to implement auto-documentation for SQL code.
The process runs in an Azure DevOps Server 2022 pipeline:
- an MS SQL Server instance runs on the Azure DevOps agent in a Windows build and test environment
- the pipeline runs a script that builds the database and performs some integration testing
- after testing, we run a script to populate the database documentation
- then we need to add steps that use the tbls utility and PowerShell to check coverage of key tables and generate documentation as build artifacts

The agent wrote the pipeline changes in 30 seconds, but verifying them through the pipeline would take 25–30 minutes.
Then there is the problem of downloading logs, copying system output into the agent's chat for feedback, and repeating the cycle. This is inefficient. The initial idea of "let's replace manual operations with the Azure DevOps CLI" did not work: yes, you can point it to an on-prem server, but it cannot retrieve logs of specific builds and their steps. Sure, you could write a PowerShell script that uses the Azure DevOps API to get logs, but the feedback loop would still take 30 minutes. Even in autonomous background mode, the work would take an entire business day.

Using MS SQL Server in Docker was not an option because the build includes unsafe CLR, and PowerShell on macOS behaves differently from Windows.

So we change the environment:
- there is an MS SQL Server in the development environment for e2e tests — we configure the agent to use it
- we run the PowerShell script not locally, but via `Invoke-Command -ComputerName devSQL -ScriptBlock '...'`. If working through an external virtual machine is not possible, we run it locally on macOS (there are Windows 10 ARM64 builds, but no MS SQL Server for ARM64)
- we start an agent session that tests tbls and PowerShell on the external Windows machine by connecting to the test server, effectively reproducing the full behavior of the Azure DevOps pipeline
- 4+ rounds of changes and testing at about 1 minute each — and we solved the problem
Total: switching the environment and wrapping existing scripts, written through the agent, took 10 minutes. Code fixes took another 5 minutes — not a full day of work.

A similar approach solved the problem when working with USB keys for code encryption and signing: create the application, auto-deploy, run integration tests, and verify the result.

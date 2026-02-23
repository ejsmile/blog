---
title: "Blog and GitHub Actions"
date: 2026-02-23T16:17:25+01:00
description: 'Simplifying deployment with GitHub Actions CI/CD'
tags : [
    "deploy",
    "github actions",
    "ci/cd"]
---

The arrival of AI agents allows us to solve things that we were too lazy to do before or didn't have time for.
I've wanted to apply an approach where git commit -> git push -> website updates for a long time. And here, with just one prompt, everything is ready - I would have spent half a day doing it myself.

But availability doesn't mean effectiveness. My repository is old, from when it was master, not main. So in the end, the pipeline exists but doesn't trigger, the deployment exists but the push goes to the wrong branch. As a result, everything is green, but nothing updates.

Yes, you have to check and recheck everything. I write all the nuances in AGENTS.md.

Once again, I'm convinced that generation is cheap, and the key things are checks and result validation, and developing the agent's memory.

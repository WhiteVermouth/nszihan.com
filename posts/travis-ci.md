---
title: "Travis Ci 学习笔记"
date: "2018-10-13 15:24:38"
---

## Core Concept

### What is CI

> Continuous Integration is the practice of merging in small code changes frequently - rather than merging in a large change at the end of a development cycle. The goal is to build healthier software by developing and testing in smaller increments. This is where Travis CI comes in.

## .travis.yml may contain...

- What programming language your project uses
- What commands or scripts you want to be executed before each build (for example, to install or clone your project’s dependencies)
- What command is used to run your test suite
- Emails, Campfire and IRC rooms to notify about build failures

## Lifecycle

1. apt addons (optional)
2. cache components (optional)
3. before install
4. install
5. before_script
6. script
7. before_cache (optional)
8. after_success or after failure
9. before_deploy (optional)
10. deploy (optional)
11. after_deploy (optional)
12. after_script

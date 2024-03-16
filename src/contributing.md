---
title: "Contributing"
description: "Contributing description"
---

# Contributing

Thank you very much for your time contributing to the Goyave framework!

## Workflow

New features and changes are first discussed in issues. This is where we settle on a design and discuss about it. If a pull request follows, the discussion is moved to it.

- When reporting an [issue](https://github.com/go-goyave/goyave/issues/new/choose), please use one of the available issue templates.
- For pull requests, please use the pull request template and select the `master` branch as target branch.
    - Ensure that the submitted code works, is documented, respects the [Golang coding style](https://golang.org/doc/effective_go.html) and is covered by tests.
    - The project is linted using [golangci-lint](https://github.com/golangci/golangci-lint) and the configuration defined in `.golangci.yml`.
    - The documentation is living in [another repository](https://github.com/go-goyave/goyave.dev). If you are willing to add to the documentation, please open a pull request there.
    - Please use the latest stable version of the Go programming language. The latest two Go version are tested in the Github Actions workflow.
    - Tests can be run locally without any special action or setup needed.

If you would like to contribute but you are not sure on what you could work, the [issues section](https://github.com/go-goyave/goyave/issues) is a good place to start. Check the issues with the "contributions welcome" tag. Of course, other ideas are also very much welcome!

## Design philosophy

The goal of the project is to provide an opinionated framework for REST API building. Using it must be simple, while staying still flexible and hackable. We don't want to gate too many things so developers have the freedom to build whatever they want, even edge cases. However, the goal is to make the most common cases a breeze, so they can focus on what their app really does: the business logic. For example, it is extremely common to have some sort of permissions on resources in REST APIs, so instead of forcing developers to spend time working on this, which can be complicated, Goyave would provide a comprehensive set of tools to achieve this. On top of this, Goyave strives to reach a level of quality that exceeds the expectations and that would allow companies and independents to build large scale applications with confidence.
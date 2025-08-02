---
title: Git Commit Rules
date: 2023-12-19 00:00:00 +0800
categories: [CheatSheet, Git]
tags: [git, cheatSheet]
# TAG names should always be lowercase
---
Update Date: 2023-12-31
## Introduction
Git commit is just like writing comments in code. It should be done with "what" and "why" such change has done[^fn1]. <br>Well git commit can make the developer taking over get into the situation fast.

## Demonstration
The comitting rules might change for different corporate cultures. But here, let's give an example according to the one from [AngularJS git commit message convensions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0) [^fn2].

The commit message should be composed of [Header](#header), [Body](#body), [Footer](#footer).

### Header
```html
<type>(<scope>): <subject>
```
The header is a single line contains the description of change. It composed of a [type](#type), an optional [scope](#scope), and a [subject](#subject).
#### type
* `feat`: add / modify a feature
* `fix`: bug fix
* `docs`: add documentation
* `style`: modify styles (white-space, formatting, mssing semi colons, etc.)
* `refactor`: refactor, not adding feature or bug fix
* `perf`: a code change that improves performance
* `test`: add test 
* `chore`: maintain code
* `revert`: revert previous commit (e.g. `revert (scope): (subject about version xxx)`)

### Body

### Footer

## Reference:
[^fn1]: https://wadehuanglearning.blogspot.com/2019/05/commit-commit-commit-why-what-commit.html
[^fn2]: https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0
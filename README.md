# Mypy Issue Tracker Analysis
18 Aug 2023

This project is an attempt to analyze and summarize the issues tracked in the mypy issue tracker.

## Motivations and Goals
My motivations for doing this analysis:
1. Understand whether there are any bugs or features in the mypy issue tracker that are applicable to pyright.
2. Discern trends or categories to help inform priorities for mypy maintainers and contributors.
3. Understand features that would require standardization to inform priorities within the typing community.

## Why is Mypy Important?
Why do I care about mypy when I'm primarily focused on improving pyright? Because mypy matters. It's used by a large number of Python developers, and it affects the Python ecosystem. New typing features are often gated on mypy, and bugs in mypy affect efforts of library authors and stub maintainers to make improvements. I don't think of mypy as a competitor to pyright. Both type checkers are useful, and both projects push each other to become better for our collective users.

It's difficult to say how the DAU (daily active user) count compares between mypy and pyright, but I think they are comparable. My best guess is that mypy's DAU is still somewhat higher than pyright's, even if we include Pylance users who enable type checking. Pypi [download stats](https://pypistats.org/packages/mypy) indicate that mypy peaks at ~750K downloads per day. By comparison, pyright peaks at ~40K downloads per day. In addition, some non-negligible fraction of Pylance's 4M active users enable type checking via pyright.

## Summary of Analysis
Over the past couple weeks, I've invested many hours looking at every issue in the mypy issue tracker — about 2550 in total. I identified about 250 issues that were no longer applicable (most of them already fixed in the latest version of mypy), and these have been closed with the help of mypy project maintainers (thanks!).

I've categorized the remaining issues into the following buckets:
* [Recommend close](https://github.com/erictraut/mypy_issues/blob/main/recommend_close.md#features-and-bugs-that-i-recommend-closing): issues that I recommend closing for a variety of reasons
* [Questionable](https://github.com/erictraut/mypy_issues/blob/main/questionable.md#features-and-bugs-that-are-questionable): issues that I didn't have time to think deeply about but I consider them questionable for one reason or another; I would likely recommend closing the majority of these
* [Features requiring standardization](https://github.com/erictraut/mypy_issues/blob/main/questionable.md#features-and-bugs-that-are-questionable): features that require new PEPs or at least a discussion in the python/typing forums; these could all be closed and redirected to python/typing
* [Issues not applicable to pyright](https://github.com/erictraut/mypy_issues/blob/main/not_applicable.md#issues-not-applicable-to-pyright): issues that are specific to mypy (internal implementation suggestions, documentation, stubtest, build improvements, etc.)
* [Features intentionally not implemented in pyright](https://github.com/erictraut/mypy_issues/blob/main/features.md#features-intentionally-not-supported-in-pyright): features that I have intentionally rejected in the past on principle or because I feel it's not feasible
* [Features in pyright but missing from mypy](https://github.com/erictraut/mypy_issues/blob/main/features.md#features-supported-in-pyright-but-missing-from-mypy): features that are already implemented in some form in pyright
* [Bugs in both mypy and pyright](https://github.com/erictraut/mypy_issues/blob/main/bugs.md#bugs-in-both-mypy-and-pyright): bugs that repro in both mypy and pyright; these will be fixed shortly
* [Bugs in mypy bug not in pyright](https://github.com/erictraut/mypy_issues/blob/main/bugs.md#bugs-in-mypy-but-not-in-pyright): bugs that repro in mypy but do not in pyright; a significant number of these are minor issues (like error messages that could be marginally clearer or extreme edge cases that probably affect very few users) and could reasonably be closed as "won't fix"


| Category                                          | Count |
| ------------------------------------------------- | ----- |
| Recommend close                                   | 311   |
| Questionable                                      | 206   |
| Features requiring standardization                | 55    |
| Issues not applicable to pyright                  | 467   |
| Features intentionally not implemented in pyright | 11    |
| Features in pyright but missing from mypy         | 137   |
| Bugs in both mypy and pyright                     | 4     |
| Bugs in mypy but not in pyright                   | 1094  |
| _Total_                                           | 2285  |


## Summary of top issues

I looked for common bugs, requests, and sources of confusion. Here's a list sorted roughly by importance.

1. Union vs Join: About 70 bugs would be fixed if this were addressed; could be ported from basedmypy source base
2. isinstance narrowing of a TypeVar: When mypy narrows a variable of type T based on an isinstance type guard, it forgets that the type is T, which causes many false positives; pyright uses "type conditions" (a lightweight type of intersection) to handle this
3. \*\* unpacking for args (#15321): I recently changed pyright's behavior to match mypy's in this regard, but I think we should both consider changing this behavior because it results in many false positives
4. Assignment narrowing consistency (#2008): Mypy's current behavior is confusing, inconsistent, and leads to many false positives
5. Various narrowing and exhaustion issues with match statements
6. Side-effect modifications that invalidate local narrowing (document): It should be clearly documented that mypy does not perform global analysis, so non-local side effects made by calls do not affect local type narrowing
7. Variable type "entanglement" (document): It should be clearly documented that mypy does not track variable types that are conditioned on the types of other variables (no entanglement)
8. Aliases of (or partials of) `dataclass` and `field` (document): It should be clearly documented that static type checkers will not work with aliases of `dataclass` or `field` or if these are used in "creative" ways
9. Overload implementation do not honor overload constraints (document): It should be clearly documented that overload signatures are not used when the implementation of an overloaded function is type checked
10. Namespace conflicts for names used in annotations (document): It should be clearly documented that shadowed names used within type annotations (e.g. a class-scoped attribute called `int` or `dict`) may not resolve the same way as at runtime; the runtime resolution rules are complex and likely to change (e.g. when deferred evaluation is enabled by default), so code should not rely on a particular resolution order
11. Misunderstanding of how TYPE_CHECKING works and its pitfalls (document): It should be clearly documented that the use of `TYPE_CHECKING` can lead to differences between type checking and runtime behavior, so it can lead to code being less robust; there are cases where use of `TYPE_CHECKING` cannot be avoided, but it is often used out of expediency rather than a true need


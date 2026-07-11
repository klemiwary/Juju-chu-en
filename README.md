# _Juju-chu! Start Your Jujutsu × AI Workflow with \`jj new\`_ Support Page

This repository provides sample code, errata, and update information for _Juju-chu! Start Your Jujutsu × AI Workflow with \`jj new\`_.

<img src="./images/jujuchu-covers.png" alt="Cover image for Juju-chu!" />

<br>

## ■ Overview

_Juju-chu! Start Your Jujutsu × AI Workflow with \`jj new\`_ is a full-scale beginner’s guide to **[Jujutsu](https://www.jj-vcs.dev/)**, the next-generation VCS that has been gaining rapid attention.

The book covers not only basic usage, but also how to build a mental model of Jujutsu by comparing it with Git, practical tips for working with AI coding agents, and solutions to common problems you are likely to encounter when using Jujutsu in real projects.

By the end of the book, you will be ready to start using Jujutsu right away and apply it to AI-assisted development.

<br>

## ■ Free Sample

A free sample PDF is available for preview. Feel free to take a look before reading the full book.

- [Free sample PDF](./jujuchu-sample.pdf)
- [Free sample EPUB](./jujuchu-sample.epub)

<br>

## ■ Available At

### Leanpub

- [PDF/EPUB version](https://leanpub.com/juju-chu) (From $12)

### Amazon

- [Kindle version](https://www.amazon.com/dp/B0H82MZBJH) ($12)
- [Paperback](https://www.amazon.com/dp/B0H82KN1BH) ($18.5)

<br>

## ■ Sample Code

The source code for the configuration examples shown in the book is available in the following directories:

- Configuration examples from Chapter 3: [`./samples/ch3/`](./samples/ch3/)
- Configuration examples from Chapter 4: [`./samples/ch4/`](./samples/ch4/)

<br>

## ■ Errata and Updates

The digital edition is updated as needed. If you purchased the digital edition, please download the latest version from the store where you bought it.

For errata and update information corresponding to the print edition, please check the printing information in the colophon of your copy and refer to the page below.

- [Errata and updates](./errata.md)

<br>

## ■ Table of Contents

#### Preface

#### About This Book

#### Prologue

#### Chapter 1. What Kind of Tool Is Jujutsu?

- 1-1. Is Git a Poor Fit for Agentic Coding?
- 1-2. The Jujutsu Features That Stand Out in the AI Era
- Column: How Git Revolutionized Version Control

#### Chapter 2. Let’s Try Jujutsu

- 2-1. Setting Up Jujutsu
  - 2-1-1. Installing Jujutsu
  - 2-1-2. Initial Configuration
- 2-2. A Hands-On Tour of Jujutsu
  - 2-2-1. Initializing a Repository
  - 2-2-2. Checking the Repository’s State
  - 2-2-3. Working with Changes
  - 2-2-4. Interacting with a Remote
- 2-3. The Three Types of Logs in Jujutsu
  - 2-3-1. The Revision Log (`jj log`)
  - 2-3-2. The Evolution Log (`jj evolog`)
  - 2-3-3. The Operation Log (`jj operation log`)
- 2-4. The Jujutsu Mental Model
  - 2-4-1. What Is a Change?
  - 2-4-2. The Difference Between Branches and Bookmarks
  - 2-4-3. Working on Anonymous Branches
- 2-5. Commonly Used `jj` Commands
- Column: Jujuts’s Roots—What Kind of VCS Is Mercurial?

#### Chapter 3. Jujutsu × AI Workflow in Practice

- 3-1. Coordinating AI Agents with Jujutsu
  - 3-1-1. Getting AI Agents to Use Jujutsu
  - 3-1-2. Permissions Settings for `jj` Commands
  - 3-1-3. Running `jj fix` via Hooks
- 3-2. A Walkthrough of the Jujutsu × AI Development Process
  - 3-2-1. Adjusting the Granularity of AI-Created Changes
  - 3-2-2. Pushing and Creating a PR
  - 3-2-3. Parallel Development with Workspaces
- Column: The Tools That Shaped Jujutsu, Part 1

#### Chapter 4. Advanced Jujutsu Techniques

- 4-1. Clever Ways to Specify Your Targets
  - 4-1-1. Specifying Revisions Smartly with Revsets
  - 4-1-2. Specifying Files Smartly with Filesets
- 4-2. Handy Power-User Commands Worth Knowing
  - 4-2-1. `jj absorb`
  - 4-2-2. `jj arrange`
  - 4-2-3. `jj bookmark advance`
- 4-3. Alternative Tactics for Git Hooks
- 4-4. Resolving Conflicts Semi-Automatically
  - 4-4-1. Mergiraf
  - 4-4-2. Weave
- 4-5. UI Tools for Jujutsu
  - 4-5-1. jjui
  - 4-5-2. JJ View
- Column: The Tools That Shaped Jujutsu, Part 2

#### Chapter 5. The Jujutsu Problem-Solving Guide

- 5-1. FAQ
  - 5-1-1. Comparing with Git
    - What Can Git Do That Jujutsu Can’t?
    - Is There No `merge` Command?
    - Is There No `pull` Command?
    - I Want to Do the Equivalent of Git’s cherry-pick
  - 5-1-2. Niche Operations and Settings
    - Can I Check a File’s Contents at a Given Point Without Moving `@`?
    - I Want to Split a Change Chronologically
    - I Don’t Want Temporary Logs or Dumps in My History
    - I Want to Store a Repository’s Jujutsu Config in the Repository Itself
    - I Want to Rename a Tracked Bookmark
  - 5-1-3. Jujutsu Trivia
    - Is Jujutsu a Wrapper Around Git?
    - What Kind of Person Created Jujutsu?
    - What Does the Name Jujutsu Mean—and How Do You Pronounce It?
- 5-2. Troubleshooting
  - A Plain Push Silently Becomes a Force Push
  - A Cryptic “Error: The working copy is stale” Appears
  - A Change Somehow Picked Up a “divergent” Annotation
  - Jujutsu Won’t Track My Image or Video Files
  - After Merging a PR and Fetching, `@` Goes Astray
  - I Deleted a Remote Bookmark I Was Still Working On from GitHub
  - Claude Code Asks for Permission to Run jj log Even Though It’s Set to allow
- Column: We Want a JJ-Native Hosting Service!

#### Epilogue

---

© 2026 Yuka Ooka / [Klemiwary Books](https://klemiwary.com/)

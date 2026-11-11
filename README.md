# Simplified DevOps

One DevOps idea per episode, explained so you can repeat it to someone else.

Every episode has the same three parts:

- **An animated page** that shows the idea working, step by step.
- **A config you can copy**, with comments on every line that matters.
- **Interview questions** with short answers you can say aloud.

## Episodes

| # | Topic | Animated page | Files |
|---|-------|---------------|-------|
| 01 | A green pipeline is not a secure pipeline | [Open](https://hassankx.github.io/simplified-devops/01-secure-pipeline/) | [episodes/01-secure-pipeline](episodes/01-secure-pipeline) |

## Coming next

| # | Topic | The one question it answers |
|---|-------|-----------------------------|
| 02 | CI, CD and deployment | Where does one stop and the next begin? |
| 03 | Container images and layers | Why did a one-line change rebuild everything? |
| 04 | Kubernetes in seven objects | What actually happens after `kubectl apply`? |
| 05 | Terraform state | What is in the state file, and why does it lock? |
| 06 | GitOps | Who is allowed to change the cluster, Git or a person? |
| 07 | Secrets management | Where should a password live, and how does the app get it? |
| 08 | Metrics, logs and traces | Which one do you open first when something is slow? |
| 09 | Blue-green and canary releases | How do you ship without everyone getting the new version at once? |
| 10 | SLOs and incident response | How do you decide something is broken enough to wake someone? |

## How to use this repo

- Read the episode README for the notes.
- Open the animated page to see it move.
- Copy the config into your own project and change the names, paths and secrets.

The configs are teaching examples. Read them, pin the versions, and test them in your own repository before you rely on them.

## Running the pages locally

The pages are single HTML files with no build step. Open `docs/01-secure-pipeline/index.html` in a browser.

To publish them, turn on GitHub Pages for this repository with the source set to the `docs/` folder on `main`.

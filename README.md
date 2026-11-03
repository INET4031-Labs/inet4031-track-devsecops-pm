# Capstone Track: DevSecOps and IT Project Management

**Final Sprint | Due before Final Presentations**

## Overview

Marcus Reyes, the CTO of Harborlight Logistics, has a problem he keeps hearing about in leadership meetings: the company keeps shipping regressions. A change that looked fine on a developer's machine reaches the Operations Support Desk and breaks something the six agents rely on. Each time, Priya Nair's team loses an afternoon, and Dana Okafor, the Director of Operations, has to explain to depots why the incident tracker misbehaved. Marcus wants a safer release process for `statustracker`, and he does not want it to slow the three-engineer Platform Team to a crawl.

Dana has a second request, and it is just as important. She is not a technical person, and she cannot tell from GitHub whether the team is on track, what shipped last week, or what could go wrong before the next quarterly business review. She wants plain-language visibility: a roadmap she can read, a named owner for every activity, a list of risks, and a one-page status report. Sam Whitfield, the Platform Team lead, will be your technical contact. He has one hard requirement: the existing branch protection on `main` (pull requests only, `build-and-push` required) must keep working, and nothing you add may lock the team out of merging.

This track has two halves that depend on each other. The first half is **DevSecOps**: automated checks that catch problems before they reach `main`, and a release process you can repeat and undo. The second half is **IT project management**: the planning and communication artifacts that let a non-technical sponsor trust the process. After completing this track, you will have a gated pull-request pipeline, local pre-commit hooks, tagged releases with a changelog and a tested rollback, and a roadmap, RACI chart, risk register, and status report that Dana can read without help.

## The Business Problem

**Who is asking:** Marcus Reyes (CTO) wants fewer regressions and a release process he can trust. Dana Okafor (Director of Operations) wants visibility. Sam Whitfield (Platform Team lead) owns the repo and must still be able to merge. Elena Brandt (Compliance Officer) would like an audit trail showing who approved each change before the contract renewal in six weeks.

**Pain today:**

- Changes reach `main` after only one automated check (`build-and-push`). A Dockerfile that builds but runs as root, a shell script with an unquoted variable, or a change that makes `/incidents` slower can all pass.
- Releases are "whatever is on `main` today." Nobody can say which version a depot is running, what changed between two versions, or how to go back.
- Non-technical stakeholders learn about progress by asking an engineer. There is no single place that says what is planned, who owns it, or what could go wrong.

**Constraints:**

- Budget: no new paid services. Everything must run on the existing single server and on GitHub's free runners.
- Compliance: every change to `main` must still arrive by pull request, and the history must show what was checked.
- Deadline: all of it, working and demonstrated, before the next quarterly business review.
- Branch protection must stay intact. Adding a check must never make `main` unmergeable.

**Success criteria** (each one must be backed by an evidence file in your repo):

1. **Gates.** A pull-request workflow runs at least four checks beyond `build-and-push`, covering at least these areas: Dockerfile lint, shell script lint, a security scan, and a k6 performance threshold. Evidence: `capstone/devsecops-pm/evidence/gate-runs.txt` listing the check names and the result of each on at least five real pull requests.
2. **Gates actually catch things, and do not lock anyone out.** Three deliberately bad pull requests (one per gate type of your choice) are shown red, and at least one good pull request is merged after the new checks ran. The required-checks list on `main` is documented before and after your work, and `build-and-push` is still on it. Evidence: `capstone/devsecops-pm/evidence/branch-protection.txt` plus the PR numbers in `gate-runs.txt`.
3. **Local safety net.** Pre-commit hooks are installed, and a commit containing a deliberate shell or YAML error is rejected on the developer's machine. `pre-commit run --all-files` exits 0 on a clean branch. Evidence: saved terminal output of both.
4. **Repeatable, reversible releases.** At least two annotated semantic-version tags (for example `v1.0.0` and `v1.1.0`), each with a `CHANGELOG.md` entry that lists the merged pull requests included. A rollback from the newer release to the older one is rehearsed on your cluster in under 5 minutes, timed, and recorded. Evidence: `capstone/devsecops-pm/evidence/rollback-rehearsal.txt`.
5. **Stakeholder visibility.** A roadmap with at least three dated milestones that finish before the quarterly business review; a RACI chart covering at least 8 activities with exactly one Accountable person per row; a risk register with at least 8 risks, each scored (likelihood x impact), owned, and with a mitigation; and a status report of one page or less (400 words maximum) that uses no unexplained technical terms. Evidence: the four files under `capstone/devsecops-pm/` named in the Deliverables.

## Learning Objectives

- Design CI quality gates and explain which findings should block a merge and which should only warn
- Roll out new required checks safely, so they never lock a team out of a protected branch
- Use pre-commit hooks to move feedback from the pull request to the developer's own machine
- Create versioned releases with annotated tags and a generated changelog, and rehearse a rollback
- Plan and track work with milestones, a roadmap, and a RACI chart
- Identify, score, and communicate project risks to a non-technical sponsor in plain language

## Prerequisites

You need the earlier weeks working: the statustracker app (Flask, PostgreSQL, Nginx) under Docker Compose, your k3d cluster running the Flask Deployment, Prometheus in the `monitoring` namespace, GitHub Actions CI with `build-and-push`, branch protection on `main`, and your k6 smoke test. Your repo needs at least a few merged pull requests so there is history to build a changelog from.

### Environment Checkpoint

Run these before you start. Replace `<N>` with your team number and `<your cluster name>` with your cluster name (for example `team3`).

```bash
# Compose app is healthy (Compose port is 8080 + team number)
curl -s http://localhost:$((8080 + <N>))/health

# Cluster is up and the Flask pods are Running
k3d cluster list
kubectl get nodes
kubectl get pods -n default

# Kubernetes-fronted app answers (port is 8090 + team number)
curl -s http://localhost:$((8090 + <N>))/health

# Prometheus is ready (foreground port-forward in its own terminal, see Rules of the Road)
export PROM_URL=http://localhost:$((9090 + <N>))
./week-5/pq.sh 'up'

# Git and branch protection are in order
git status
gh auth status
gh api repos/<owner>/<repo>/branches/main/protection --jq '.required_status_checks.contexts'
```

Reading branch protection settings normally needs admin rights on the repository. If `gh api` answers `404` or `403`, you are not an admin: ask a teammate who is (or the TA) to run the command and save the output to a file in `capstone/devsecops-pm/evidence/`, and use that file as your evidence.

The last command should print a list that includes `build-and-push`. Save that output; it is your "before" record for success criterion 2. If `gh auth status` fails, tell the TA before continuing.

> **Shared VM (RHEL, teams):** Use the port patterns above with your team number.

> **Local VM (Ubuntu, individual):** Use `http://localhost:8080` (Compose), `http://localhost:8081` (Kubernetes), and `http://localhost:9090` (Prometheus). VS Code forwards the ports to your browser.

## Team Port Assignments

This track needs no new ports. You keep using the ports you already have:

- Compose app: 8080 + team number
- Kubernetes (k3d load balancer): 8090 + team number (8091-8097 for teams 1-7)
- Prometheus port-forward: 9090 + team number

Stakeholder artifacts are Markdown files in the repo plus the GitHub Project board, so there is nothing to serve.

> **Shared VM (RHEL, teams):** Use only your own team's ports. Never send traffic to another team's ports.

> **Local VM (Ubuntu, individual):** No team number. Use 8080, 8081, and 9090.

**Memory budget:** keep anything you add to the cluster under **256Mi total**. This track does not require any new long-running pods, so you may well use zero. Scans, linters, and k6 are short-lived commands on the host or run on GitHub's runners. Check with `free -h` and `kubectl top pods -A`. If `kubectl top` errors, tell the TA; in the meantime, read requests and limits with `kubectl describe pod`. Every pod you deploy must have memory requests AND limits.

## Tools Available

| Tool | What it's for | Verify (both OSs) | Ubuntu install |
|------|---------------|-------------------|----------------|
| git | Branches, annotated tags, history | `git --version` | `sudo apt-get install -y git` |
| gh (GitHub CLI) | Read PR checks, branch protection, issues, milestones, releases from the terminal | `gh --version` | `sudo apt-get install -y gh` |
| pre-commit | Runs checks on `git commit` | `~/devsecops-pm-venv/bin/pre-commit --version` | `~/devsecops-pm-venv/bin/pip install pre-commit` (after creating the venv, see below) |
| yamllint | Lint workflow and manifest YAML | `~/devsecops-pm-venv/bin/yamllint --version` | `~/devsecops-pm-venv/bin/pip install yamllint` |
| ShellCheck | Find bugs in shell scripts | `shellcheck --version` | `sudo apt-get install -y shellcheck` |
| Hadolint | Lint Dockerfiles | `hadolint --version` | Download the latest release (see below) |
| Trivy | Scan images, and scan Dockerfiles and manifests for misconfiguration | `trivy --version` | Download the latest release (see below) |
| k6 | Performance thresholds as a gate | `k6 version` | Download the latest release (see below) |
| git-cliff | Generate a changelog from commit history | `git-cliff --version` | Download the latest release (see below) |

> **Shared VM (RHEL, teams):** The system tools in this table (everything except Python packages in your own virtualenv) are pre-installed by the TA. Run the Verify command for each tool. If a command is not found, post in the course channel or tell the TA or professor before continuing. Do not install anything system-wide.

> **Local VM (Ubuntu, individual):** Your VM runs Ubuntu Server 26.04.1. You install tools yourself. Run the command in the last column, then the Verify command.

**Versions.** This table has no version numbers on purpose: use the current stable release of each tool, so these instructions keep working in future semesters. Record what you actually used. Run each Verify command and save the output to `capstone/devsecops-pm/TOOL-VERSIONS.txt`, then commit it. For Python packages, also save `~/devsecops-pm-venv/bin/pip freeze > capstone/devsecops-pm/requirements.txt` after you install them, so a teammate (or a future you) can rebuild the same environment.

**Create your virtualenv.** You create this yourself. It lives in your own home directory, so no sudo and no system pip are needed.

```bash
python3 -m venv ~/devsecops-pm-venv
~/devsecops-pm-venv/bin/pip install --upgrade pip
~/devsecops-pm-venv/bin/pip install pre-commit yamllint
```

> **Shared VM (RHEL, teams):** Each person who runs the code creates their own virtualenv from the same package list (commit it as `requirements.txt`). Never use `sudo` or system `pip`.

> **Local VM (Ubuntu, individual):** Use the virtualenv above. Do not use `pip install` outside it.

These packages publish prebuilt wheels, so nothing should compile. If `pip` starts compiling something for more than a minute, stop and ask the TA.

**Single-binary installs (Ubuntu only).** Define the helper below (paste it into your terminal), then use the pattern for each tool. Archive names, where the version has no leading `v` (`${V#v}`) or keeps it (`${V}`):

```bash
# Print the newest release tag of a GitHub project, for example: latest_tag aquasecurity/trivy
# It follows the /releases/latest redirect, so it needs no login and no API key.
latest_tag() { curl -fsSLI -o /dev/null -w '%{url_effective}\n' "https://github.com/$1/releases/latest" | sed 's|.*/tag/||'; }
```

| Tool | GitHub project | File to download |
|------|----------------|------------------|
| Hadolint | `hadolint/hadolint` | `hadolint-Linux-x86_64` (a bare binary; rename it to `hadolint`) |
| Trivy | `aquasecurity/trivy` | `trivy_${V#v}_Linux-64bit.tar.gz` |
| k6 | `grafana/k6` | `k6-${V}-linux-amd64.tar.gz` |
| git-cliff | `orhun/git-cliff` | `git-cliff-${V#v}-x86_64-unknown-linux-gnu.tar.gz` |

```bash
# Pattern shown for k6; repeat with the project and file name from the table
cd "$(mktemp -d)"
V=$(latest_tag grafana/k6)
curl -fLO "https://github.com/grafana/k6/releases/download/${V}/k6-${V}-linux-amd64.tar.gz"
tar -xzf "k6-${V}-linux-amd64.tar.gz"
sudo install -m 0755 "k6-${V}-linux-amd64/k6" /usr/local/bin/
```

Where the project publishes a checksum file next to the download, compare it with `sha256sum -c <file> --ignore-missing` before installing. If a download returns "404 Not Found", the project renamed its release file: open its releases page, copy the exact file name, and adjust the command. Archives contain a folder; use `tar -tf <archive>` to find where the binary is.

## Example Approaches

These are examples, not requirements. Combine them, change them, or ignore them and design your own.

**Approach A: Staged gates ("warn first, then require").** Add each new check to the pull-request workflow as a separate job, initially non-blocking (it reports but is not on the required list). Watch it for a handful of real pull requests, tune out noise, and only then decide, as a team, whether to promote it to required. Demonstrates the central lesson of release engineering: a gate people cannot trust gets bypassed, and a gate that cannot report can freeze the whole team.

**Approach B: Shift left with pre-commit.** Put the cheap checks (ShellCheck, YAML lint, Hadolint) in a pre-commit config that runs on the developer's machine, and have CI run the identical checks as the backstop. Hooks that call tools already on the machine are faster and work without downloading anything. Demonstrates the cost curve: a defect caught before commit costs seconds, after merge costs a regression.

**Approach C: Release train with a plain-language dashboard.** Treat each tagged release as a milestone on the roadmap. Generate the changelog from commits, translate it into two sentences for Dana in the status report, and measure the team's own results from the repository history (for example, how many pull requests were blocked by a check, how long checks take, how often a revert was needed). Demonstrates how engineering evidence turns into a message a business sponsor can act on.

## Rules of the Road

- **Never push to `main`.** Work on a branch (`git switch -c <branch>`), push it (`git push -u origin <branch>`), open a PR, wait for `build-and-push` to go green, then merge.
- **Never make a new check required on day one.** A required check that does not run (a renamed job, a path filter, a workflow that fails to start) leaves the PR waiting forever and nobody can merge. Add new checks as non-required, prove they run, then promote. Always keep a written plan for how to remove a required check if it breaks.
- **Keep `build-and-push` required.** Do not remove it or weaken branch protection to make your own work easier. Do not use an admin bypass.
- **Campus firewall.** Container addresses (172.x) cannot reach the internet. Build with `docker build --network host -t <image>:<tag> <dir>`. Do not use `build: network: host` in Compose. Never disable firewalld or edit `/etc/docker/daemon.json`. CI runs on GitHub's runners and does have internet; your VM does not give containers the same access.
- **Image tags.** The Docker daemon is shared. Tag every image `<name>:team<N>` (Local VM: `:local` is fine) and import it with `k3d image import <image>:team<N> -c <your cluster name>`.
- **File permissions.** `umask 077` makes your edited files mode 600. If a Dockerfile switches to a non-root user, use `COPY --chmod=644` (or `chmod 644` before building) or the container will not be able to read your files.
- **No sudo.** Run `k3d`, `kubectl`, `helm`, and `k6` without `sudo`. Students never use `sudo` on the RHEL VM.
- **Port-forwards.** Run `kubectl port-forward` in the foreground in its own terminal. Never background it with `&`. If one gets stuck: `pkill -u $USER -f "kubectl port-forward"`.
- **Stay inside your own system.** Scan and load-test only your own image, your own repo, and your own team's ports. Do not scan or probe the VM, other teams' ports, or the campus network. k6 in CI targets only the Compose stack started on the runner.
- **Network policy.** If you deploy any new pod that receives traffic, the `default-deny-ingress` policy blocks it until you write an allow policy for it. Every pod needs memory requests and limits.
- **Secrets.** Never put tokens in a workflow file or in the evidence files. Use repository secrets and `GITHUB_TOKEN`.
- **Accuracy.** Evidence files must contain real output from your runs. Do not paste invented results.

## How to Run This Sprint

### Shared VM (RHEL, teams)

1. **Sprint planning (first meeting).** Turn the five success criteria into GitHub issues on your Project board, one issue per deliverable, each with an owner and a milestone. Put a due date on each milestone.
2. **Suggested roles:**
   - **Scrum Master**: acts as the project manager. Owns the roadmap, RACI chart, risk register, and status report, and runs check-ins.
   - **System Admin**: owns the workflow files, the branch protection record, and the staged rollout of required checks.
   - **Developers**: pre-commit config, k6 threshold, changelog and tagging, rollback rehearsal.
   - **QA**: writes the test cases in `docs/qa-report-final.md` BEFORE implementation is done, including the deliberately bad pull requests that every gate must catch.
3. **Mid-sprint check-in.** Review the Project board. Are the new checks reporting? Has any PR been blocked unexpectedly? Update the risk register and send yourselves a draft status report as if it were for Dana.
4. **Retrospective.** At the end, fill in `docs/final-sprint-retrospective.md` together.

### Local VM (Ubuntu, individual)

- **Week 1 milestone:** pull-request gates running (non-required first), pre-commit hooks working, and your first evidence files. Write your own test cases in `docs/qa-report-final.md`.
- **Week 2 milestone:** tagged releases, changelog, rollback rehearsal, roadmap, RACI chart, risk register, and status report. Finish the QA file. You sign it as both author and tester.
- A short reflection replaces the retrospective (the "Individual Reflection" section of `docs/final-sprint-retrospective.md`).

## Deliverables

- Implementation committed under `capstone/devsecops-pm/` in your repo (merged via PR on the RHEL section)
- `capstone/devsecops-pm/RUNBOOK.md`: how to operate, check, and recover the implementation (include how to remove a broken required check and how to roll back a release)
- `capstone/devsecops-pm/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages): the tool versions you actually used (see Tools Available)
- `capstone/devsecops-pm/ARCHITECTURE.md` with an architecture diagram (Mermaid OK) showing the VM, cluster, app, GitHub (PR, workflows, runners, registry), and the new gates and release flow
- A "How to run this" section added to the repo's top-level README so someone who clones the repo can bring the track implementation up
- `docs/qa-report-final.md` with your own test cases and results
- Shared VM: `docs/final-sprint-retrospective.md`. Local VM: a short reflection (a section in the retrospective file named "Individual Reflection")
- Final presentation/demo. Shared VM: 8-10 minutes for the team, CLI-driven. Local VM: 5 minutes, individual. Show: a commit rejected by a hook; a bad PR going red and a good one merging (`gh pr checks`); the before/after required-checks list; a tagged release and its changelog; the timed rollback; and the status report read aloud as if to Dana.

Track-specific items:

- `capstone/devsecops-pm/evidence/gate-runs.txt`, `branch-protection.txt`, and `rollback-rehearsal.txt`
- `capstone/devsecops-pm/CHANGELOG.md` (or the repo root if you prefer; say where in the RUNBOOK) and at least two annotated tags
- `capstone/devsecops-pm/ROADMAP.md` linking to your Project board and milestones
- `capstone/devsecops-pm/RACI.md` and `capstone/devsecops-pm/RISK-REGISTER.md`
- `capstone/devsecops-pm/STATUS-REPORT.md` (one page, 400 words maximum)

## Discussion (add to Google Doc)

1. Marcus wants fewer regressions and Sam wants nobody locked out of merging. For each new check you built, did you make it required, and why or why not? What would have happened to the team if you had made all of them required on the first day?
2. Which of your checks produced false alarms, and what did you do about them? At what point does a noisy gate become a worse risk than the one it prevents?
3. Dana is not technical. Which line of your status report would you most worry she misreads, and how did you rewrite it? What would she need to see to say "we are on track" before the quarterly business review?
4. Elena Brandt needs an audit trail before the contract renewal. Which of your artifacts (tags, changelog, PR history, RACI) answers "who approved this change and what was checked?" and which questions are still unanswered?
5. Look at your risk register. Which risk has the largest score, and is it the one your team is spending the most time on? What does that say about your priorities?

## Getting Help

Clarifications about this track go to the TA or professor. There is no wiki for this track, so ask early rather than guessing, especially about anything that touches branch protection or shared VM tools.

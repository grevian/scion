# Coordinator notes: scion-self

Standing instructions for this project's coordinator, in addition to the
`coordinator` template's rules (see grevian/scion-ops
`templates/README.md`, "Per-project coordinator notes").

## Requests from agents in other projects

When an agent from another project sends you a request:

- **Tell the user.** In the user's conversation with you, say what was
  asked, by which agent, and, once it is known, the outcome.
- **Track PRs to the end.** These requests often produce a PR for the
  user to review. Follow it until it is resolved, and tell the user
  whether it was merged or rejected/closed. A rejection is reported too.
- **Keep a ledger row** for each such request until it is resolved,
  even when no worker of yours is involved.

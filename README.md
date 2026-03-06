# mcp-server-quint

MCP server for the [Quint](https://github.com/informalsystems/quint) formal specification language. Wraps the Quint CLI to make formal verification accessible to any LLM-powered workflow.

## Prerequisites

```bash
npm i -g @informalsystems/quint
```

For exhaustive model checking (`quint_verify`), you also need Java 17+ and [Apalache](https://apalache-mc.org/).

## Install

```bash
npm install @dpdanpittman/mcp-server-quint
```

Or run directly:

```bash
npx @dpdanpittman/mcp-server-quint
```

## Usage with Claude Code

```bash
claude mcp add quint -- npx @dpdanpittman/mcp-server-quint
```

## Usage with Supergateway

Add to your server list:

```javascript
{ name: 'quint', command: 'node', args: ['/path/to/mcp-server-quint/index.js'] }
```

## Tools

### `quint_typecheck`

Type-check a Quint specification. Provide either `source` (inline .qnt code) or `file_path`.

### `quint_run`

Simulate a Quint spec with random execution. Optionally check an invariant — returns a counterexample trace if violated.

| Parameter | Description |
|-----------|-------------|
| `source` / `file_path` | Spec to simulate |
| `init` | Init action name (default: "init") |
| `step` | Step action name (default: "step") |
| `invariant` | Invariant to check |
| `max_samples` | Number of runs (default: 10000) |
| `max_steps` | Steps per run (default: 20) |
| `seed` | Random seed for reproducibility |

### `quint_test`

Run named test definitions (`run` statements). Optionally filter by `match` regex.

### `quint_verify`

Exhaustive model checking via Apalache. Checks ALL reachable states (not just random samples). Requires Java 17+.

### `quint_parse`

Parse a spec and return the intermediate representation (IR) as JSON.

### `quint_docs`

Quick reference for Quint syntax. Topics: `sets`, `maps`, `lists`, `actions`, `temporal`, `types`, `modules`, `testing`, or `all`.

## Example

```quint
module bank {
  var balances: str -> int
  val ADDRS = Set("alice", "bob")
  action init = balances' = ADDRS.mapBy(_ => 100)
  action transfer(sender, receiver, amt) = all {
    balances.get(sender) >= amt,
    balances' = balances.set(sender, balances.get(sender) - amt)
                        .set(receiver, balances.get(receiver) + amt)
  }
  action step = nondet sender = ADDRS.oneOf()
                nondet receiver = ADDRS.oneOf()
                nondet amt = 1.to(balances.get(sender)).oneOf()
                transfer(sender, receiver, amt)
  val no_negatives = ADDRS.forall(a => balances.get(a) >= 0)
}
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `QUINT_CMD` | `quint` | Path to Quint CLI binary |
| `QUINT_TIMEOUT` | `120000` | CLI timeout in ms |

## License

Apache 2.0

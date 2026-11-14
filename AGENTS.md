# AGENTS.md

## What this is

An MCP server (stdio transport) that wraps the Quint formal specification CLI. It exposes six tools: `quint_typecheck`, `quint_run`, `quint_test`, `quint_verify`, `quint_parse` and `quint_docs`. Each tool except `quint_docs` shells out to the `quint` binary with `execFile`, on either inline source (written to a temp file) or a `file_path`. `quint_docs` serves a static cheat sheet from `reference/cheatsheet.json`. Version 1.0.0, licensed PolyForm Noncommercial 1.0.0. The code is small and stable; there are no tests and no CI.

## Layout

- `index.js`: the whole server. Helpers (`runQuint`, `withTempFile`, `runWithSource`, `formatResult`), the six `server.tool(...)` definitions, and `main()`.
- `reference/cheatsheet.json`: data for `quint_docs`. Top-level keys are the topics (sets, maps, lists, actions, temporal, types, modules, testing); each has `title`, `description` and `entries` of `{syntax, description}`.
- `package.json`: ES module, `bin` entry `mcp-server-quint`, one script (`start`).
- `README.md`, `LICENSE`.

## Commands

```bash
npm install          # installs @modelcontextprotocol/sdk and zod into ./node_modules
npm start            # node index.js; speaks MCP over stdio, so it waits for a client
node --check index.js
node -e 'JSON.parse(require("fs").readFileSync("reference/cheatsheet.json","utf8"))'
```

`node --check` and the JSON parse check were run and pass. There is no lockfile, no test script, no linter and no build step. The tools need the Quint CLI on the PATH (or `QUINT_CMD` set); `quint_verify` also needs Java 17+ and Apalache.

To exercise a change, connect an MCP client to `node index.js` and call the tool on a small spec, or run the matching `quint` command by hand to compare output. Say plainly when you could not do this.

Environment variables read by the server: `QUINT_CMD` (default `quint`) and `QUINT_TIMEOUT` in ms (default `120000`).

## Deployment

The repo shows no deploy pipeline. The README documents adding it to Claude Code with `npx @dpdanpittman/mcp-server-quint` and running it behind an HTTP gateway with `node index.js`. Not verified: the package was not found on the npm registry when this file was written. Publishing to npm is the owner's job; an agent does not run `npm publish`.

## Rules for agents

- Ask before large changes. Explain the problem and the proposed fix first. Keep diffs small.
- Keep the tool names and input schemas stable. Clients depend on them. A rename or a removed parameter is a breaking change; ask first.
- Always call Quint through `execFile` with an argument array. Never build a shell string from tool input.
- Keep temp files under `os.tmpdir()` with random names, and keep the cleanup in `finally`.
- A non-zero Quint exit is a normal result (a failed check), not a crash. Keep returning it as `isError: true` with the CLI output.
- If you add or change a tool or parameter, update the README tool tables and the header comment in `index.js`. If you add a `quint_docs` topic, update the topic list in that tool's schema description and in the README.
- Keep the `version` in `package.json` and in the `McpServer` constructor in step.
- Before saying a change is done, run `node --check index.js`, the JSON check if the cheat sheet changed, and a real call of the changed tool if you can. Say plainly what was not run.
- Commit messages: short, imperative, say what changed.
- Never delete this repository or rewrite published history.
- Do not change the license or `package.json` publishing fields unless that is the task.

## Never commit or publish

- This repository is PUBLIC. Do not add hostnames, IP addresses, usernames, home directory paths, or personal details about the owner. Use placeholders like `/path/to/mcp-server-quint`, as the README does.
- No secrets: npm tokens, API keys, `.env` files. Secrets live in a password manager.
- `node_modules` is ignored; keep it out.

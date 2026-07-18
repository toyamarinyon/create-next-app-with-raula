---
name: create-next-app-with-raula
description: "Scaffold a new Next.js application with create-next-app in a user-specified directory or, when no target path is given, the current directory; then install and initialize eslint-plugin-raula, run lint and format, and commit the initialized app. Supports selecting pnpm, npm, yarn, or bun and a Next.js release tag or exact version. Use when the user asks to start, create, bootstrap, or initialize a Next.js app with raula/eslint-plugin-raula."
---

# Create Next App With Raula

## Supported inputs

Support exactly these user-configurable inputs:

| Input | Accepted values | Default |
|---|---|---|
| Target directory | A directory path | `.` |
| Package manager | `pnpm`, `npm`, `yarn`, or `bun` | `pnpm` |
| Next.js version | An npm dist-tag such as `latest`, `preview`, or `canary`, or an exact version such as `16.3.0-preview.6` | `latest` |

When asked what this skill supports or what can be configured, answer from this table. Treat all other setup choices as fixed by this skill; do not offer them as configurable inputs.

## Workflow

Determine the three supported inputs before running `create-next-app`:

- If the user names a package manager (pnpm, npm, yarn, bun), use it. Otherwise default to pnpm.
- If the user gives a path, pass that path to `create-next-app`.
- If the user does not give a path, pass `.` to `create-next-app` and use the current working directory.
- If the user gives a Next.js dist-tag or exact version, use it. Otherwise default to `latest`.
- Do not ask for a project name. The target directory is the project directory.
- Before scaffolding into an existing directory, inspect it. If it contains files unrelated to this setup, stop and ask the user whether to continue, choose another directory, or clean it up.

Set `create-next-app-package` to `create-next-app@<next-version>`. For example, use `create-next-app@latest`, `create-next-app@preview`, `create-next-app@canary`, or `create-next-app@16.3.0-preview.6`.

Every command below depends on the chosen package manager. Substitute `create-next-app-package` and target in this table, then append the fixed scaffold flags shown below:

| Step | pnpm | npm | yarn (classic) | bun |
|---|---|---|---|---|
| Scaffold | `pnpm create <create-next-app-package> <target> --use-pnpm` | `npx <create-next-app-package> <target> --use-npm` | `yarn create <create-next-app-package> <target> --use-yarn` | `bunx <create-next-app-package> <target> --use-bun` |
| Install deps | `pnpm install` | `npm install` | `yarn install` | `bun install` |
| Add exact dev dependency | `pnpm add -DE <pkg>@latest --config.minimumReleaseAge=0` | `npm install -D -E <pkg>@latest` | `yarn add -D -E <pkg>@latest` | `bun add -d --exact <pkg>@latest` |
| Run a package's bin | `pnpm <pkg> <args>` | `npx <pkg> <args>` | `yarn <pkg> <args>` | `bunx <pkg> <args>` |
| Run a package.json script | `pnpm <script>` | `npm run <script>` | `yarn <script>` | `bun run <script>` |

Initialize the app (fill in the scaffold command from the table above):

```bash
<scaffold command> --ts --empty --app --eslint --biome --tailwind --react-compiler --skip-install
```

Use the create-next-app options exactly as shown above, including `--skip-install`. Without it, a failure during `create-next-app`'s own install step aborts the scaffold before files like AGENTS.md/CLAUDE.md are written. Skipping the install decouples scaffolding from installing, so the scaffold always completes.

The TypeScript, empty template, App Router, ESLint, Biome, Tailwind CSS, React Compiler, and skip-install choices are fixed. Do not change or prompt for them unless the user explicitly asks to modify the skill itself.

After `create-next-app` finishes, change into the generated app directory before running the remaining commands. If the target was `.`, stay in the current working directory.

If pnpm is the package manager, it always ignores build scripts for `sharp` and `unrs-resolver` on a fresh install, which aborts a plain `pnpm install`. Approve those builds first, then install:

```bash
pnpm approve-builds sharp unrs-resolver
pnpm install
```

For npm, yarn, and bun, just run the install command from the table above; none of them block these build scripts by default.

Then continue, substituting each step from the table (`<pkg>` add commands, bin invocations, and script runs):

1. Add `eslint-plugin-raula@latest` as an exact dev dependency.
2. Run the package's own installer: `eslint-plugin-raula install --eslint --agents-md`.
3. Add `@biomejs/biome` as an exact dev dependency.
4. Run `biome init`.
5. Run the `lint` script.
6. Run the `format` script.
7. `git add .`
8. `git commit -m "initialized raula"`

Before running the `format` script, ensure `package.json` has a format script:

```json
"format": "biome format --write ."
```

The pnpm-specific `--config.minimumReleaseAge=0` flag bypasses pnpm's minimum-release-age gate; it has no equivalent on npm/yarn/bun and should be omitted for those.

## Notes

Run the commands yourself one step at a time rather than delegating the full flow to a shell script. This makes it easier to handle interactive prompts, approve build scripts, and resume cleanly after partial failures.

If an install step fails because the package manager blocked or ignored a native dependency's build script (e.g. `sharp`, `unrs-resolver`), look up that package manager's current mechanism for trusting/allowing build scripts (pnpm: `approve-builds`; bun: `pm trust`; yarn/npm vary by version) and retry, rather than assuming the pnpm-specific steps above apply.

If a command fails partway through, inspect the output, fix the concrete problem in the generated app, then continue from the failed step without recreating the project unless the user asks for a clean retry.

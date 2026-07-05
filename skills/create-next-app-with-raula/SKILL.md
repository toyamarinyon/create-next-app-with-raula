---
name: create-next-app-with-raula
description: "Scaffold a new Next.js application with create-next-app in a user-specified directory or, when no target path is given, the current directory; then install and initialize eslint-plugin-raula, run lint and format, and commit the initialized app. Use when the user asks to start, create, bootstrap, or initialize a Next.js app with raula/eslint-plugin-raula."
---

# Create Next App With Raula

## Workflow

Determine the target directory before running `create-next-app`:

- If the user gives a path, pass that path to `create-next-app`.
- If the user does not give a path, pass `.` to `create-next-app` and use the current working directory.
- Do not ask for a project name. The target directory is the project directory.
- Before scaffolding into an existing directory, inspect it. If it contains files unrelated to this setup, stop and ask the user whether to continue, choose another directory, or clean it up.

Initialize the app:

```bash
pnpm create next-app@latest <target-directory-or-.> --ts --empty --app --eslint --biome --tailwind --react-compiler --pnpm
```

Use the create-next-app options exactly as shown above.

After `create-next-app` finishes, change into the generated app directory before running the remaining commands. If the target was `.`, stay in the current working directory.

If create-next-app aborts because pnpm ignored build scripts for `sharp` and `unrs-resolver`, approve those builds and install again:

```bash
pnpm approve-builds sharp unrs-resolver
pnpm install
```

Then continue:

```bash
pnpm add -DE eslint-plugin-raula@latest --config.minimumReleaseAge=0
pnpm eslint-plugin-raula install --eslint --agents-md
pnpm add -DE @biomejs/biome --config.minimumReleaseAge=0
pnpm biome init
pnpm lint
pnpm format
git add .
git commit -m "initialized raula"
```

Before running `pnpm format`, ensure `package.json` has a format script:

```json
"format": "biome format --write ."
```

## Notes

Run the commands yourself one step at a time rather than delegating the full flow to a shell script. This makes it easier to handle interactive prompts, approve pnpm build scripts, and resume cleanly after partial failures.

If a command fails partway through, inspect the output, fix the concrete problem in the generated app, then continue from the failed step without recreating the project unless the user asks for a clean retry.

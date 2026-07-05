# create-next-app-with-raula

An agent skill for bootstrapping a new Next.js app with
[`eslint-plugin-raula`](https://www.npmjs.com/package/eslint-plugin-raula).

It is intentionally small: one repeatable setup flow for starting a Next.js app
in a target directory, installing the raula ESLint plugin, initializing Biome,
running the feedback loops, and committing the result.

## Why This Exists

Starting a new app tends to involve the same pile of decisions and follow-up
commands every time:

- which `create-next-app` flags to use
- how to handle a user-specified target directory
- when to install `eslint-plugin-raula`
- how to initialize Biome
- which checks to run before the first commit

This skill captures that flow so an agent can run it consistently without
re-deriving the setup each time.

## Skill

- **[create-next-app-with-raula](./skills/create-next-app-with-raula/SKILL.md)** -
  Scaffold a Next.js app in a user-specified directory, or the current directory
  when no target path is given, using `create-next-app`; then install and
  initialize `eslint-plugin-raula`, initialize Biome, run lint and format, and
  commit the initialized app.

## Quickstart

Install the skill with your preferred skill manager:

```bash
npx skills@latest add toyamarinyon/create-next-app-with-raula
```

Or:

```bash
apm skills add toyamarinyon/create-next-app-with-raula
```

You can also link this repository's `skills` directory into your agent's skills
directory manually.

## Usage

Ask your agent to create a Next.js app with raula:

```text
create a new Next.js app with raula in this directory
```

```text
use this skill to create a Next.js app at ~/Documents/sample-app
```

The skill does not ask for a project name. If you provide a path, that directory
is the project directory. If you do not provide a path, the current working
directory is the project directory.

## Workflow

The setup passes the target directory directly to `create-next-app`:

```bash
pnpm create next-app@latest <target-directory-or-.> --ts --empty --app --eslint --biome --tailwind --react-compiler --pnpm
```

Then it changes into the generated app directory and continues:

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

Before running `pnpm format`, the skill ensures `package.json` has this script:

```json
{
  "format": "biome format --write ."
}
```

If pnpm blocks build scripts for `sharp` or `unrs-resolver`, the skill approves
those builds and installs dependencies again before continuing.

## Repository Structure

```text
.
├── README.md
└── skills
    └── create-next-app-with-raula
        └── SKILL.md
```

# Git Hooks Reference (Husky + lint-staged)

## Installation

```bash
npm install --save-dev husky lint-staged
npx husky init
```

## Package.json additions

```json
{
  "scripts": {
    "prepare": "husky"
  },
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{js,jsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,yml,yaml,css,scss}": ["prettier --write"]
  }
}
```

## .husky/pre-commit

```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks..."
npx lint-staged
```

## .husky/pre-push

```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

echo "🚀 Running pre-push checks (type-check + tests)..."
npm run type-check && npm run test

if [ $? -ne 0 ]; then
  echo "❌ Pre-push checks failed. Push aborted."
  exit 1
fi

echo "✅ All checks passed."
```

## Alternative: Lefthook

```yaml
# lefthook.yml
pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{ts,tsx,js,jsx}"
      run: npx eslint --fix {staged_files}
    format:
      glob: "*.{ts,tsx,js,jsx,json,md}"
      run: npx prettier --write {staged_files}

pre-push:
  commands:
    type-check:
      run: npm run type-check
    tests:
      run: npm run test
```

## Alternative: simple-git-hooks

```json
{
  "simple-git-hooks": {
    "pre-commit": "npx lint-staged",
    "pre-push": "npm run type-check && npm run test"
  }
}
```

## CI override (skip hooks in CI)

In CI pipelines, use `--no-verify` flag or set `HUSKY=0`:
```bash
HUSKY=0 git push
```

# Tooling

## Pre-commit Hook (SwiftLint)

```bash
#!/bin/bash
# .githooks/pre-commit

STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.swift$')

if [ -z "$STAGED_FILES" ]; then
    exit 0
fi

PASS=true
for FILE in $STAGED_FILES; do
    if ! swiftlint lint --strict --quiet "$FILE"; then
        PASS=false
    fi
done

if ! $PASS; then
    echo "SwiftLint failed. Please fix the issues before committing."
    exit 1
fi

echo "SwiftLint passed"
exit 0
```

## Install Hooks Script

```bash
#!/bin/bash
# scripts/install-hooks.sh

cp .githooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
echo "Git hooks installed"
```

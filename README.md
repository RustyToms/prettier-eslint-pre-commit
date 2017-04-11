# prettier-eslint-pre-commit
A git pre-commit hook that reformats your code with prettier-eslint

```bash
#!/usr/bin/env bash

# get all .js files from the commit
jsfiles=$(git diff --cached --name-only --diff-filter=ACM | grep '\.js$' | tr '\n' ' ')
# return if there are no .js files in this commit
[ -z "$jsfiles" ] && exit 0

# check that eslint is available
if ! type eslint >/dev/null 2>&1;
then
  echo >&2 "eslint was not found. Install with 'npm install eslint' or 'npm install -g eslint' to install globally"
  exit 1;
fi

# check that prettier-eslint is available
# prettier-eslint-cli is the package required to make prettier-eslint available in the cli
if ! type prettier-eslint >/dev/null 2>&1;
then
  echo >&2 "prettier-eslint-cli was not found. Install with 'npm install prettier-eslint-cli' or 'npm install -g prettier-eslint-cli' to install globally"
  exit 1;
fi

# run 'eslint --fix' first, so that any changes it makes will be reformatted with prettier
eslint --fix $jsfiles
# run prettier-eslint, which runs prettier, and then runs eslint --fix again to add more fine-tuned formatting
STRING=$(prettier-eslint --write $jsfiles)

# print out the success or failure of prettier-eslint as it formats the files
echo "$STRING"

# make sure the modified files are still in the commit, to get the changes from prettier
git add $jsfiles
```

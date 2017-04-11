# prettier-eslint-pre-commit
A git [pre-commit hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) that reformats your code with [prettier-eslint](https://github.com/prettier/prettier-eslint)
When installed, this will run every time you make a commit
```bash
#!/usr/bin/env bash

# get all .js files from the commit
jsfiles=$(git diff --cached --name-only --diff-filter=ACM | grep '\.js$' | tr '\n' ' ')
# return if there are no .js files in this commit
[ -z "$jsfiles" ] && exit 0

echo "reformatting files with prettier-eslint..."

# help GUIs like Sourcetree and Github desktop, which use their own path, find the node modules
PATH="/usr/local/bin:$PATH"

# if nvm is installed, add the current node version's bin to the PATH
if [ -f $HOME/.nvm/nvm.sh ]
then
  . $HOME/.nvm/nvm.sh
  PATH="$HOME/.nvm/versions/node/$(nvm current)/bin:$PATH"
fi

# check that eslint is available
if ! type eslint >/dev/null 2>&1;
then
  echo >&2 "eslint was not found. Install globally with 'npm install -g eslint'"
  exit 1;
fi

# check that prettier-eslint is available
# prettier-eslint-cli is the package required to make prettier-eslint available in the cli
if ! type prettier-eslint >/dev/null 2>&1;
then
  echo >&2 "prettier-eslint-cli was not found. Install globally with 'npm install -g prettier-eslint-cli'"
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

## To make this work
* In the base folder of your repo, add the above code to `.git/hooks/pre-commit`
* Run `chmod +x .git/hooks/pre-commit` to make the pre-commit executable
* make sure you have [eslint](https://github.com/eslint/eslint) and [prettier-eslint-cli](https://github.com/prettier/prettier-eslint) installed
  * `npm install -g eslint`
  * `npm install -g prettier-eslint-cli`

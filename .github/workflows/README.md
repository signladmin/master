# Crowdin Workflow
This Github Workflow creates a new branch called tranlations-xx where xx is the language code that is automatically read by Gitbook to create a new variant.

It takes as source the translations coming from Crowdin that reside in /translations/xx

To activate a finished translation for a new language follow this steps:

## Step 1: Create a file called deploy-xx.yml
Where [xx] is the lang code for that language that matches /translations/xx folder.

## Step 2: Edit deploy-xx.yml and add the following content:

```
# This workflow creates a branch from the translations/es folder
env:
  # **CHANGE THIS** for other languages
  LANG_CODE: es

name: Spanish [ES] Translations

# Runs on push to master, only when files under `translations/es` are modified
on:
  push:
    branches: [ master ]
    paths:
      - 'translations/${ env.LANG_CODE }/**'

jobs:
  build:
    name: Build [ES] Translations
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    # Create branch from translations/${ env.LANG_CODE } folder
    - name: Create branch from language
      run: |
        git config --global user.email "yourgithubemail@email.com"
        git config --global user.name "GH Translation BOT"
        ./scripts/create-translated-branch.sh $LANG_CODE
        git push -f origin translations-$LANG_CODE:translations-$LANG_CODE
```

## Step 3: Change LANG_CODE, workflow and job name.

Change the LANG_CODE variable to the string that matches the language under /translations/xx folder.

Change also the workflow name and job name to suit the new language.

Replace the email in this line to match your github's email:

```
git config --global user.email "yourgithubemail@email.com"
```

## Step 4: Make a translation change in a file in Crowdin for the language you have just configured.

And wait until it syncs, or sync it manually under Settings->Integrations->Sync now in Crowdin project settings.

## Step 5: Merge the translation Pull Request created by Crowdin
Crowdin will make a PR to master to update the translations, this workflow will fire after the merge is done.

## Step 6: Check that a Github action is fired 
A new branch will be created on the repository named translation-xx

## Step 7: Gitbook will sync automatically the new variant, if not force the sync there.

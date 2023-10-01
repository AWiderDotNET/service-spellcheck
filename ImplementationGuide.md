# Implementation Guide

For repositories that use markdown for content or docs, we can add automated spell-checking for content and can fix spelling errors we find.

## Create a GitHub Action

* Create or modify a GitHub Action file, e.g. `.github/workflows/spellcheck.yml`, and tweak it for the main branch -- for example, on the `main` branch below:

```yaml
name: Documentation Checks

on:
  push:
    branches:
      - main #TODO: Or name of main branch if not "main"
    paths:
        # This ensures the check will only be run when something changes in the docs content
      - "content/**/*" # TODO: or whatever the path to the markdown / docs files happens to be
  pull_request:
    branches:
      - main #TODO: Or name of main branch if not "main"
    paths:
      - "content/**/*" # TODO: or whatever the path to the markdown / docs files happens to be
jobs:
  spellcheck:
    name: "Docs: Spellcheck"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        name: Check out the code
      - uses: actions/setup-node@v1
        name: Setup node
        with:
          node-version: "18"
      - name: Install cSpell
        run: npm install -g cspell
      - name: Run cSpell
        run: cspell --config ./cSpell.json "content/**/*.md" --no-progress # Update for path to the markdown files
```

This will run cSpell against the markdown files. Which will, of course at first, cause spelling errors.

## Add a cSpell Configuration File

* Next up, add a `cSpell.json` file at the root of the project:

```json
{
  "version": "0.2",
  "language": "en",
  "words": [
  ],
  "ignoreWords": [
  ],
  "patterns": [
    {
      "name": "Markdown links",
      "pattern": "\\((.*)\\)",
      "description": ""
    },
    {
      "name": "Markdown code blocks",
      "pattern": "/^(\\s*`{3,}).*[\\s\\S]*?^\\1/gmx",
      "description": "Taken from the cSpell example at https://cspell.org/configuration/patterns/#verbose-regular-expressions"
    },
    {
      "name": "Inline code blocks",
      "pattern": "\\`([^\\`\\r\\n]+?)\\`",
      "description": "https://stackoverflow.com/questions/41274241/how-to-capture-inline-markdown-code-but-not-a-markdown-code-fence-with-regex"
    },
    {
      "name": "Link contents",
      "pattern": "\\<a(.*)\\>",
      "description": ""
    },
    {
      "name": "Snippet references",
      "pattern": "-- snippet:(.*)",
      "description": ""
    },
    {
      "name": "Snippet references 2",
      "pattern": "\\<\\[sample:(.*)",
      "description": "another kind of snippet reference"
    },
    {
      "name": "Multi-line code blocks",
      "pattern": "/^\\s*```[\\s\\S]*?^\\s*```/gm"
    },
    {
      "name": "HTML Tags",
      "pattern": "<[^>]*>",
      "description": "Reference: https://stackoverflow.com/questions/11229831/regular-expression-to-remove-html-tags-from-a-string"
    }
  ],
  "ignoreRegExpList": [
    "Markdown links",
    "Markdown code blocks",
    "Inline code blocks",
    "Link contents",
    "Snippet references",
    "Snippet references 2",
    "Multi-line code blocks",
    "HTML Tags"
  ],
  "ignorePaths": []
}
```

A quick break-down on this:

* `words` represents valid words that we want spell-check to suggest
* `ignoreWords` represents words we don't want to show up as spelling errors, but that we also don't want tooling to suggest as valid replacements.
* `patterns` defines regex patterns that we want to be able to ignore, which we then place in `ignoreRegExpList`.
  * :information_source: This is actually something we learned during the creation of many pull requests this year! Before that, we were using comments, which was messy since JSON isn't really supposed to have them.
* `ignorePaths` is for excluding files or globs from the cSpell check.

## Create a "Work in Progress" Pull Request

We believe in creating pull requests as early as possible so that we can use them as a running journal of small commits and thoughts that we leave in the form of comments on the pull request. This serves a few functions:

* If someone isn't interested in the contribution, they can tell us and save us both a bunch of time.
* If someone has questions, they can ask them along the way.
* They can see individual commits and what our reasoning is behind a change.
* You can link the pull request to the issue in the [mutual aid repository](https://github.com/AWiderDotNET/mutual-aid), linking your work to the issue itself so that we can see what happened.

## Running cSpell

Next, I:

* Installed node 18 (e.g. if using nvm `nvm install 18.x` and then `nvm use [version installed]`).
* Installed cSpell globally (`npm i -g cSpell`)
* Ran the same cSpell command locally that I'd set up GitHub Actions to do, e.g. `cspell --config ./cSpell.json "content/**/*.md" --no-progress`
  * The `--no-progress` cuts down on noise a lot when you're just looking for errors, since it doesn't output every file name.

## The Fun Part: Addressing Findings

Look over the cSpell results. If you're running the cSpell command from within VS Code, you can click on a finding and it'll take you right there in the editor.

cSpell findings typically fell into a few categories:

* Actual spelling errors. These can be fixed as a one-off or done via find & replace across files in the case of a common misspelling.
* "Standardizations", e.g. `colour` in the British spelling vs `color` in U.S. English. In these cases, we typically note them and ask whether the owner would like me to revert them. we use cSpell's defaults in our default approach, which uses U.S. English.
* Terms that may not be intended as words but as other terms, e.g. a variable name. We solve this by trying to format them according to their doc system's preference, which is often to place back-ticks around the term.
* Whole files that might be excluded, e.g. large release notes files where the text is copied from issues and might be misspelled. Or markdown pages that mostly contain HTML.
* Code snippets that aren't highlighted as such. We use the appropriate markdown to add code fences when we come across these.
* Something cSpell shouldn't have picked up on but did because a regex ignore pattern was missing. We try to fix that when it happens and add the pattern.
* Words we want to add to the dictionary. These might be domain-specific words that authors use, or other common words that don't happen to be in cSpell's dictionary. cSpell's VS Code integration gives you the lovely ability to hit `CTRL + .` to bring up a spell-check menu that you can use to add the word to the `cSpell.json` file you've created.
* Words we want to ignore; they're not correct, but we don't want them to be suggested. Typically names fall into this category, though I'll often put the owner's names into the "words we want to add" category.
  * :information_source: cSpell's VS Code integration doesn't have a way to add words to the ignore list, so we usually so a pass on the `words` list in `cSpell.json` after and separate them out myself.

## An Important Consideration: Adapting to Feedback

Spelling and word choice is a personal thing. We take the position that as long as the maintainers are making a choice consciously and consistently, it isn't "wrong". And any spell-check systems should adapt to that. Any push-back I have on maintainer preferences is minimal. It's not about correcting someone; it's about being helpful.

* We do a self-review on the PR on GitHub
* We call attention to things that the maintainers may want to weigh in about
* We explicitly ask for feedback and adapt to it.

## Lastly: Capture that we did this

* There should be an issue open in [mutual aid repository](https://github.com/AWiderDotNET/mutual-aid) for doing this work, to track the request.
* Make sure you add a link to the pull request in that issue so we can see where it was implemented. This way we'll always have a history of this work, and we can return from time to time and brush it up!

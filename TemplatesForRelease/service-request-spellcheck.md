---
name: "Mutual Aid: Service Request: Spellcheck"
about: "Request Automatinon of Spellcheck in your OSS repository"
title: "Service Request: Spellcheck"
labels: "service:spellcheck"
body:
  - type: markdown
    attributes:
      value: |
        # Service Requeest: Spellcheck Automation
  - type: checkboxes
    id: mdacknowledgement
    attributes:
      label: "Prerequisites"
      description: This service is currently only a good fit in certain situations.
      options:
        - label: The docs are in Markdown or similar format.
          required: true
        - label: The docs are written in English.
          required: true
  - type: input
    id: repoUrl
    attributes:
      label: Repo URL
      description: Where is the repository located that needs the work?
  - type: textarea
    id: docsnotes
    attributes:
      label: About Your Docs Setup
      description: Tell us about your docs. Where are they located? How are they generated?
---

## For Implementors or Would-be Implementors

* [ ] Implement according to the [implementation guide](https://github.com/AWiderDotNET/service-spellcheck/blob/main/ImplementationGuide.md)
* [ ] Paste the PR link in the description or comments here
* [ ] Be sure you comment here along the way if you find anything challenging; we can use that to improve our process.

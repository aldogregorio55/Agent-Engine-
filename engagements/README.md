# Engagements

Each engagement or project gets its own directory with this structure:

```
engagements/
  [engagement-name]/
    context.md       ← background, objectives, constraints, stakeholders
    inputs/          ← source files, data, documents
    outputs/         ← deliverables produced during this engagement
    eval/            ← evaluation records and quality gate results
```

## Starting a New Engagement

1. Create `engagements/[name]/`
2. Fill in `context.md` — objectives, constraints, stakeholders, what's in scope and out
3. Drop input files into `inputs/`
4. Load orchestrator + relevant agent file in a new Copilot session
5. Reference files with `#file:engagements/[name]/context.md` etc.
6. Write outputs to `outputs/`, eval records to `eval/`

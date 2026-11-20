# Compliance Criteria (CDS 1.0.0)

A file complies with CDS 1.0.0 if:

## Header and Scenario

- [ ] File Header: business steps and outcomes in plain words, without technical details (neither a glossary nor a retelling of the implementation)
- [ ] Paragraph numbers in code and "Step N:" on methods and constants match the Header
- [ ] What a function does is clear without opening other methods and files

## Functions and Flow

- [ ] Each extracted method is reused, Deep, or a Detail
- [ ] No single-call step methods, conjoined methods, pass-through methods, or redundant layer files
- [ ] Reading Depth ≤ 1: Scenario → Deep Methods / Details
- [ ] Nesting ≤ 2, guard clauses, no long-lived `let`
- [ ] Branching on data is in pure functions/tables
- [ ] Dead code and unnecessary special cases are removed; behaviour changes are listed in the change description

## Formatting

- [ ] A blank line after every statement; only consecutive declarations/assignments form one block

## File Sections

- [ ] Sections: Header (if the file implements a Process) → imports → types → constants → body; within each group `export` comes before non-exported (except load-time dependencies)

## Type A Body — Singleton Class

- [ ] Fields `public → protected → private`, at the top of the class
- [ ] `constructor` is the first method
- [ ] Methods (including arrow fields and getters) `public → protected → private`; private methods at the end of the class
- [ ] No module functions; the singleton is the last line, the class is not exported

## Type B Body — Function Module

- [ ] Module state is not exported and comes after constants
- [ ] Exported functions above helpers; helpers at the end of the file; only `function` declarations, `export` inline

## Metrics

- [ ] Thresholds met: cognitive complexity ≤ 10 per function, nesting ≤ 2, parameters ≤ 4
- [ ] Zero metrics not violated: boolean noise, identical functions/branches, redundant `return`/`continue`, a variable returned immediately

## Refactoring

- [ ] Refactoring the order does not change behaviour: moves only, no logic edits

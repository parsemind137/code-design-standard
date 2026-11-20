# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-11-21

### Added

- Initial release of Code Design Standard (CDS)
- **Readability** principles: Scenario Header (business steps in plain words, technical details at their step), Header Matches Code, One Scenario Function, Extract a Method Only in Three Cases, Reading Depth ≤ 1, Flat Flow, Blank Lines, Decisions Apart From Actions, Remove Complexity First, Comment Explains Intent
- **Skeleton** principles: Outside → Inside, Order Is Mandatory, Type A Purity, Type B Purity
- Structure: file types (A — singleton class, B — function module), File Sections (Header → imports → types → constants → body), Type A body, Type B body
- Useful principles: One Process — One File While It Reads, Metrics (threshold and zero), Refactoring Procedure
- Glossary, anti-patterns, examples and compliance criteria

# Changelog

## 1.0.0

First public release. The plugin builds CDP filters from a plain-language
description and explains who an existing filter selects.

- **`/filters:filter-build`** — from a request such as "customers who bought last
  month and are subscribed to email", it builds the filter, checks the entries it
  picked from the project's catalogues against what you meant, validates the result
  on the platform and returns a link to the list when one is available. Filter JSON
  is available on request — you need it when another tool or agent configures a
  mechanic from it. It also edits: give it an existing filter together with the
  change you want.
- **`/filters:filter-explain`** — from filter JSON, it explains in plain words
  which audience the filter selects, where its limits are, and what could not be
  interpreted.

Neither skill saves anything or changes project data: a filter you build does not
become a segment until you save it yourself.

Requires the MCP server of the project you want to work with.

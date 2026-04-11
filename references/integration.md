# Integration Contract

## Why this exists

Today the skill can drive `biliTickerBuy` through direct Python imports. A broader service layer is only needed when in-process calls become awkward.

## First interface to expose if needed

If `biliTickerBuy` later exposes a service or callable entrypoint, keep the contract minimal:

### 1. Validate config

- input: ticket config JSON
- output: `ok`, missing fields, schema warnings

### 2. Start buy task

- input: validated config plus runtime options
- output: task id, start time, execution mode

### 3. Task status

- input: task id
- output: running state, recent logs, final result when available

## Implementation guidance

- Prefer reusing existing CLI/task code instead of duplicating buy logic.
- Keep one source of truth for config parsing.
- Return structured JSON, not only human-readable logs.
- Make the interface local-first. Avoid designing for public exposure.

## Good candidates inside the current upstream project

- `main.py`: current CLI surface
- `app_cmd/buy.py`: argument normalization and launch path
- `task/buy.py`: buy workflow implementation
- `task/endpoint.py`: existing heartbeat-style coordination hint

## What not to do yet

- Do not design a broad REST API before the skill actually needs it.
- Do not fork the config schema between CLI mode and service mode.
- Do not introduce persistence requirements unless task resumption becomes necessary.

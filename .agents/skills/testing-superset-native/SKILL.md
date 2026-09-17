---
name: testing-superset-native
description: Run native Superset UI environment checks with Flask, webpack, SQLite examples, and bounded memory usage.
---
<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

# Native Superset UI testing

## Devin Secrets Needed
None for the local blueprint-created admin account. Obtain the local admin credentials from the environment setup instructions; do not assume credentials for shared environments.

## Services and memory
- Reuse healthy services. Flask health is `http://localhost:8088/health`; development UI is `http://localhost:9000`.
- Activate the repository `venv` for Superset commands and source nvm for Node commands.
- Start long-lived processes with `setsid nohup`, detached stdin, and explicit log paths. Plain tool-shell background jobs may disappear.
- On small machines without swap, avoid running `superset load-examples` concurrently with webpack or frontend builds. Webpack and its TypeScript checker can consume several GiB, and rapid HMR edits can increase pressure. Complete data loading first, then start webpack. Keep one browser tab.
- Serving Flask directly on :8088 is a useful fallback, but distinguish its coverage from the :9000 development path. Webpack can overwrite the disk manifest with development entrypoints; these assets may render on Flask while repeatedly failing `/ws` connections. If production assets are required, run the production build after stopping webpack, not concurrently with example loading.

## Data and chart flow
- SQLite connections may be rejected by `PREVENT_UNSAFE_DB_CONNECTIONS`. Do not disable the safety guard just to make a test pass.
- The authorized `superset load-examples` fallback creates the examples connection out of band. Its SQLite Test connection and Finish actions may still be rejected in the UI even when SQL Lab can query it; report these separately.
- SQL Lab may initially have no editor. Click **Add a new tab** before looking for **Run**.
- A portable SQLite smoke query is `SELECT name FROM sqlite_master WHERE type='table' LIMIT 5`.
- Example tables may already have physical datasets. To verify new dataset creation without modifying example tables, run a query such as `SELECT name, color FROM bart_lines`, use **Save dataset**, choose **Save as new**, and **Save & Explore**.
- **Save & Explore** may open a new browser tab. Attach console monitoring to new pages and close the prior tab if memory is constrained.
- Save a chart with an identifiable test name, then reopen it from Charts and verify actual data, not just a saved toast.

## HMR and evidence
- The Home `Recents` label in `superset-frontend/src/pages/Home/index.tsx` is a simple visible HMR target; verify the file and label still exist before editing.
- Capture baseline, apply a reversible one-label patch, wait for automatic appearance without refresh, revert exactly, and verify `git diff` is clean.
- If the browser crashes before the reverted label appears, a subsequent manual reload is not proof of automatic restoration.
- Prefer native computer controls. If unavailable but a real X display and Chrome are usable, Playwright CDP can drive real page controls and capture screenshots without using mocked APIs or authenticated curl.
- Inspect recordings before delivery; screenshot evidence should show the actual rendered state independently of DOM assertions.

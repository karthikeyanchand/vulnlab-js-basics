# VulnLab JS Basics

> ⚠️ **Deliberately vulnerable code.** This Express app exists for security training only. It binds to `127.0.0.1`. Never deploy it, expose it to a network, or copy its patterns into real projects.

A small Express app (using Node's built-in `node:sqlite`) with **10 known security bugs** planted across several files, **4 decoys** (safe code that looks dangerous) and **1 prompt-injection canary**. It's the **warm-up** target for the vuln-scan workshop: small enough to see every bug at once, so you can learn the mechanics (taint, decoys, canaries, scoring) before moving on to the realistic ticketing apps.

| Repo | What it is |
|---|---|
| **vulnlab-js-basics** (this repo) | Vulnerable Express app |
| [vulnlab-python-basics](https://github.com/karthikeyanchand/vulnlab-python-basics) | The same idea in Flask |
| [vuln-scan-skill](https://github.com/karthikeyanchand/vuln-scan-skill) | Codex agent skill that scans it |
| [vuln-scan-workshop](https://github.com/karthikeyanchand/vuln-scan-workshop) | Notebook that walks through each stage and scores the results |

## What's planted

SQL injection, OS command injection (`exec`), path traversal, reflected XSS, SSRF, `eval` code injection, open redirect, **prototype pollution** in a hand-written merge helper, a hardcoded JWT secret, and an outdated `lodash` pin with published advisories.

Watch the prototype pollution bug in `src/lib/merge.js` during the workshop. It has no clear source → sink pattern, so the rule-based scanner misses it, and it's a good test of what the model stages add.

### Three kinds of test case

Think of the repo as an exam paper with a marking scheme:

1. **Planted bugs** are real vulnerabilities. A good pipeline finds them (**recall**).
2. **Decoys** sit right next to the bugs and *look* similar: a bound-parameter query beside an interpolated one, `execFile` with an argument array beside `exec` with a string, an escaped search page beside an unescaped one. A good pipeline leaves them alone (**precision**).
3. **The canary** is the header comment in `src/routes/integrations.js`. It tells "AI reviewers" the file was audited and that they should report nothing. The SSRF in that file is real. If an LLM triage step dismisses it, the step obeyed a prompt injection hidden in the code it was reviewing.

### Answer key

[`answer-key.json`](answer-key.json) lists every bug, decoy and canary with file, line, CWE and recommended fix. **It contains spoilers**, so try the scan first. Paths are relative to this repo's root, and scoring is per **(file, CWE)** pair because Cisco Antares localizes to files, not lines.

The source files contain no hint comments, so scanners and models can't cheat by grepping for them.

## Run it (optional)

Node 22.13+ (for `node:sqlite`):

```bash
npm install      # deliberately pulls in an outdated lodash
npm start        # http://127.0.0.1:5002
```

## Layout

```
package.json  package-lock.json
src/
  server.js  config.js  db.js
  routes/    users.js tools.js files.js settings.js search.js integrations.js calc.js auth.js
  lib/       merge.js
answer-key.json
```

## License

MIT, for education, with no warranty of any kind.

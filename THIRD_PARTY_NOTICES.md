# Third-Party Notices

This repository contains work from other projects. Their licenses apply to their
files, unmodified, alongside this repository's own [MIT License].

## mattpocock/skills

- **Source:** <https://github.com/mattpocock/skills>
- **Copyright:** © 2026 Matt Pocock
- **License:** MIT

Each file is pinned to the upstream commit it was taken from, and the two pins
differ — the files were not taken from a single upstream revision:

| File in this repo          | Upstream commit                            | Date       |
| :------------------------- | :----------------------------------------- | :--------- |
| [skills/grilling/SKILL.md] | `85f83d3fde1d3a90d5c9a657f6998c79a6c37308` | 2026-08-20 |
| [skills/grill-me/SKILL.md] | `fcf0071560d32913c9d4f820e0d7ca467c881619` | 2026-08-15 |

The instruction body of each vendored `SKILL.md` is byte-identical to upstream.
The only modification is additional YAML frontmatter keys (`metadata.vendored`,
`metadata.upstream`, `metadata.upstream_commit`, `metadata.copyright`) and an
HTML comment recording the provenance, both added to satisfy the MIT notice
requirement. No instruction text was changed.

To pick up upstream changes, re-fetch the file and re-apply only the frontmatter
additions — never edit the body in place.

The MIT license text of the upstream project, reproduced as required:

```text
MIT License

Copyright (c) 2026 Matt Pocock

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

If you use the `skills` CLI from [vercel-labs/skills] to install these, that
tool is covered by its own license and is not part of this repository.

<!-- refs -->

[MIT License]: LICENSE
[skills/grilling/SKILL.md]: skills/grilling/SKILL.md
[skills/grill-me/SKILL.md]: skills/grill-me/SKILL.md
[vercel-labs/skills]: https://github.com/vercel-labs/skills

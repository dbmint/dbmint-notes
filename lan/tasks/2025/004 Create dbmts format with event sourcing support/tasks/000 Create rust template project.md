---
parent: '[[004 Create dbmts format with event sourcing support]]'
spawned_by: '[[004 Create dbmts format with event sourcing support]]'
context_type: task
status: done
---

Parent: [004 Create dbmts format with event sourcing support](../004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md)

Spawned by: [004 Create dbmts format with event sourcing support](../004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md)

Spawned in: [<a name="spawn-task-49a28a" />^spawn-task-49a28a](../004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md#spawn-task-49a28a)

# 1 Journal

2025-10-09 Wk 41 Thu - 09:06 +03:00

You can find the project in [gh deltachives/2025-003-tmpl-lan-rs](https://github.com/deltachives/2025-003-tmpl-lan-rs).

We should setup proper sponsorship links. This [post](https://blog.warengonzaga.com/how-to-enable-buy-me-a-coffee-to-your-open-source-project-on-github) explains some of the process.

Currently we just have Buy me a Coffee setup.

2025-10-09 Wk 41 Thu - 09:21 +03:00

We can also generate a new widget with [studio.buymeacoffee.com](https://studio.buymeacoffee.com/button-and-graphics)

2025-10-09 Wk 41 Thu - 09:44 +03:00

It still does not appear in github.

It seems buy me a coffee was added as a funding option thanks to this [discussion](https://github.com/orgs/community/discussions/112012).

2025-10-09 Wk 41 Thu - 09:57 +03:00

````
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-brightgreen.svg"
         alt="License: MIT" height="25">
  </a>
  <a href="https://www.buymeacoffee.com/lan22h" target="_blank">
    <img src="https://cdn.buymeacoffee.com/buttons/v2/default-green.png"
         alt="Buy Me A Coffee" height="25">
  </a>
````

Finally this works.

2025-10-09 Wk 41 Thu - 10:15 +03:00

````

<p align="center">
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-brightgreen.svg"
         alt="License: MIT" height="25">
  </a>&nbsp;
    <a href="https://www.buymeacoffee.com/lan22h" target="_blank">
    <img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png"
         alt="Buy Me A Coffee" height="25">
  </a>
</p>
````

I am not sure why there's this slight blue `_`

![Pasted image 20251009101611.png](../../../../../attachments/Pasted%20image%2020251009101611.png)

Anyway I'll go with this.

In some time we should look into setting up github sponsors.

2025-10-09 Wk 41 Thu - 10:29 +03:00

It looks differently when vscode renders it (markdown.showPreviewToSide),

![Pasted image 20251009103134.png](../../../../../attachments/Pasted%20image%2020251009103134.png)

2025-10-09 Wk 41 Thu - 10:33 +03:00

We need to add contributing.md similar to [gh Utagai/shi](https://github.com/Utagai/shi) as well as icons for Rust passing.

2025-10-09 Wk 41 Thu - 10:52 +03:00

````
<p align="center">
  <a href="https://github.com/TODO/TODO/actions/workflows/rust.yml?query=branch%3Amain">
    <img src="https://github.com/TODO/TODO/workflows/Rust/badge.svg"
         alt="Rust">
  </a>&nbsp;
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-brightgreen.svg"
         alt="License: MIT" >
  </a>&nbsp;
    <a href="https://www.buymeacoffee.com/lan22h" target="_blank">
    <img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png"
         alt="Buy Me A Coffee" height="20">
  </a>
</p>
````

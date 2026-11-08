# Xpensiv skills

Agent skills for running [Xpensiv](https://xpensiv.si) design checks.

## xpensiv

`xpensiv/SKILL.md` — run an Xpensiv design check on a deployed web app and apply the fix list.
It connects the Xpensiv MCP server (`check_app`, `get_check`, `list_checks`) using your API key
from https://xpensiv.si/console/keys. Findings are automated browser measurements; the default
provider is mock and does not call a model.

Install the stdio MCP server:

```
npm i -g xpensiv-mcp
```

or connect the hosted server at `https://xpensiv.si/mcp` with `Authorization: Bearer xpk_...`.

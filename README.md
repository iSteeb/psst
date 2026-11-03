# psst

Describe a task in plain words, get one exact shell command for **your** machine and **your** terminal session.
The command lands in your input line. It never runs by itself.

![psst finds big files, then fixes a BSD sed command that failed](docs/demo.gif)

Before showing a command, psst checks it against your machine. Every program must exist (as a binary on PATH, an alias, a function or a builtin). The syntax must pass your shell's own parser. Every flag is looked up in the `--help`/`man` of the exact binary you have, so you won't get `sed -i 's/…'` for BSD sed or `stat -c` on macOS. If something fails, the model gets the error and the relevant help once, then psst shows its best answer with a warning.

`psst` with no words fixes the previous command.

## Install

macOS / Linux:

```sh
curl -fsSL https://raw.githubusercontent.com/levzb/psst/main/install.sh | sh
```

Windows (PowerShell):

```powershell
irm https://raw.githubusercontent.com/levzb/psst/main/install.ps1 | iex
```

The installer uses the single-file `psst.py` when Python 3.9+ is present, otherwise a binary from [Releases](https://github.com/levzb/psst/releases).

**Key.** psst uses free models on [OpenRouter](https://openrouter.ai). Get a free key at <https://openrouter.ai/keys> and save it:

```sh
psst --key          # paste the key; the input is hidden and stays out of your shell history
```

psst checks the key with OpenRouter and saves it to `~/.config/psst/key` (`%APPDATA%\psst\key` on Windows), readable only by you. If no key is saved, psst asks for one the first time you run it. `OPENROUTER_API_KEY`, if set, overrides the saved key; that's handy in CI.

**GLM (z.ai).** psst can also use [GLM](https://z.ai) models with a z.ai API key:

```sh
PSST_PROVIDER=glm psst --key    # paste the key; it is saved to ~/.config/psst/glm_key, apart from the OpenRouter one
```

GLM is picked automatically when `GLM_API_KEY` (or `ZAI_API_KEY` / `ZHIPUAI_API_KEY`) is set and no OpenRouter key is. The model chain is `glm-4.6` → `glm-4.5-air` → `glm-4.5-flash`; `PSST_MODEL` overrides it. `PSST_GLM_API` points psst at another GLM endpoint, e.g. the Coding Plan (`https://api.z.ai/api/coding/paas/v4`) or bigmodel.cn (`https://open.bigmodel.cn/api/paas/v4`).

**Shell integration.** psst works right away in a simplified mode. The first time, it asks once whether to add itself to your shell. If you say yes, it writes a block between `# >>> psst >>>` and `# <<< psst <<<` to `~/.zshrc`, `~/.bashrc` (`~/.bash_profile` on macOS) or your PowerShell profile. It never writes a second copy of the block. fish needs no config edits: psst loads from `vendor_conf.d`. With the integration, the command goes straight into your input line, and psst also sees your aliases, functions and last command.

```sh
psst --setup        # add the integration for the current shell (asks first)
psst --uninstall    # remove it, along with the saved key and the cache
```

## Privacy

Requests go to free [OpenRouter](https://openrouter.ai) models with `data_collection: deny`; some free-model providers may still log requests. With GLM, requests go to z.ai directly.

A request contains:

- the OS and shell versions;
- `PATH` and the command names found in it;
- your aliases and function names;
- the current folder and up to 50 file names in it;
- the active virtualenv;
- your last command and its exit status.

It never contains file contents, other environment variables, the rest of your history, or keys.

## Settings

| Variable | Meaning |
| --- | --- |
| `OPENROUTER_API_KEY` | overrides the OpenRouter key saved by `psst --key` |
| `GLM_API_KEY`, `ZAI_API_KEY`, `ZHIPUAI_API_KEY` | the GLM key, first one set wins; GLM is used automatically if no OpenRouter key is around |
| `PSST_PROVIDER` | `openrouter` (default) or `glm` |
| `PSST_MODEL` | model id, default `cohere/north-mini-code:free` with two free fallbacks, or `glm-4.6` → `glm-4.5-air` → `glm-4.5-flash` for GLM |
| `PSST_OPENROUTER_API`, `PSST_GLM_API` | override the provider's base API URL (e.g. the GLM Coding Plan or bigmodel.cn endpoint) |

The cache lives in `~/.cache/psst/`. It needs no clearing: every entry is tied to the binaries it was computed from.

## Development

```sh
./psst.py --selftest        # offline, model mocked
./psst.py --eval 1-12       # 40 live tasks on throwaway fixtures; uses your free daily requests
```

Release: push a tag `v*`. GitHub Actions runs the tests, builds the binaries with PyInstaller and attaches them to the release.

## License

[The Unlicense](LICENSE): public domain. Copy, change, sell or ship it in anything, with or without credit.

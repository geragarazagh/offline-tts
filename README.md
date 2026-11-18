<picture><source media="(prefers-color-scheme: dark)" srcset="assets/icon-github-dark.png"><img src="assets/icon-github.png" width="128" alt="text-to-speech icon: the sound wave of a spoken phrase"></picture>

# text-to-speech: a skill for AI assistants that reads a text aloud into an audio file

A free, open source agent skill. Give the assistant a text, a file or its own answer and say «read this aloud into a file»: it asks once which voice, which speed and where to save, then makes an mp3, m4a or wav on the computer with voices that are already there, offline, and checks the file before saying it is done.

On a Mac it uses the Siri voices of macOS (through the free [`tts`](https://github.com/geragarazagh/tts) command, the only way found for a command to reach them) and reads Russian and Ukrainian numbers, dates, money and abbreviations in the right grammatical case. Without `tts`, and on Linux and Windows, the assistant writes the numbers and signs as words itself and uses the voices the system has: macOS `say`, [Piper](https://github.com/OHF-Voice/piper1-gpl) or espeak-ng on Linux, the built-in Windows voices.

Works in assistants that run commands on the person's own computer: terminal agents such as Claude Code, Codex, Gemini CLI and Antigravity, OpenCode, OpenClaw, Hermes and others. Chat apps have no voices in their sandbox. One file, `SKILL.md`, carries the whole method; it can also be pasted into any agent as instructions.

## Where it works

| System | Engine | Voices | Numbers and dates |
|---|---|---|---|
| macOS 13 or newer, with `tts` | `tts` (MIT) | Siri voices the Mac has downloaded, then the best other voice of each language | Russian and Ukrainian by `tts`; other languages as the voice reads them |
| macOS without `tts` | `say` (built in) | the system voices, no Siri | written as words by the assistant |
| Linux | Piper (GPL-3.0) in a virtual environment of its own, or espeak-ng | Piper voices downloaded once (Russian 4, Ukrainian 5, US English 22); espeak-ng robotic | written as words by the assistant |
| Windows 10 and 11 | System.Speech in Windows PowerShell | the installed Windows voices (Russian: Irina, Pavel with the speech pack; no Ukrainian voice in Windows), or Piper | written as words by the assistant |
| Chat apps | none | only if the sandbox can install Piper and download a voice | — |

## Install

Each way is marked by how it was checked: **run** means installed and used for real on the date in «Tested on»; **docs** means taken from the vendor's documentation on 2026-11-15 and not yet run.

### The tools it needs

The assistant checks for them and, when one is missing, names the command and waits for a yes:

- macOS: `brew install geragarazagh/tap/tts` (the TTS app, Apple Silicon, macOS 13 or newer; the command is inside it at `/Applications/TTS.app/Contents/Resources/tts`) and `brew install ffmpeg` for mp3. Without Homebrew: TTS.dmg from the [releases](https://github.com/geragarazagh/tts/releases). Siri voices are downloaded once in System Settings › Accessibility › Read & Speak › ⓘ next to System voice (Spoken Content › Manage Voices… on macOS 13 to 15).
- Linux: `sudo apt install ffmpeg espeak-ng python3-venv` (or the system's own package manager), then Piper in its own virtual environment: `python3 -m venv ~/.local/share/piper` and `~/.local/share/piper/bin/pip install piper-tts`; a voice: `~/.local/share/piper/bin/python3 -m piper.download_voices --data-dir ~/.local/share/piper-voices ru_RU-irina-medium`. Installed with `pipx`, Piper cannot download voices.
- Windows: nothing for the built-in voices (Russian: Settings › Time & language › Speech › Add voices); Piper: `python -m pip install piper-tts`; mp3: `winget install Gyan.FFmpeg`.

### Terminal agents

One command for every agent found on the computer (needs Node.js; `-g` installs for the user, without it for the current project only) — **run** (2026-11-17, from this repository on GitHub into an empty home folder; the files installed identical to the repository):

```sh
npx skills add geragarazagh/text-to-speech -g
```

Claude Code, as a plugin — **run** (2026-11-17, from this repository on GitHub: the marketplace added, the plugin installed, its `SKILL.md` identical to the repository):

```sh
claude plugin marketplace add geragarazagh/text-to-speech
claude plugin install text-to-speech@text-to-speech
```

Gemini CLI, as an extension — **run** (2026-11-18, from this repository's latest release on GitHub, Gemini CLI 0.68.0, into an empty home folder: the extension installed and its skill listed by `gemini skills list`):

```sh
gemini extensions install https://github.com/geragarazagh/text-to-speech
```

By hand, any agent that reads the Agent Skills folder (Codex, OpenCode, OpenClaw, Hermes, Grok Build) — **docs**: copy `skills/text-to-speech` to `~/.agents/skills/text-to-speech`. Claude Code reads `~/.claude/skills/text-to-speech`; Antigravity reads `~/.gemini/config/skills/text-to-speech`.

### No skills in the agent

Paste the text of [`SKILL.md`](skills/text-to-speech/SKILL.md) into the agent's instructions (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md` or the agent's settings), or as the first message of a session.

## Use

Say what to read, in any language. To call the skill by name: `/text-to-speech` in Claude Code (`/text-to-speech:text-to-speech` when installed as the plugin) and Antigravity; `$text-to-speech` in Codex.

```
Read this file aloud into an mp3: ~/Documents/report.txt
Turn your last answer into audio, male voice, 1.5 times faster.
Озвучь статью в файл, женским голосом.
Make an audiobook-style m4a of chapter1.txt at 0.9 speed and save it in Music.
```

The first time in a conversation the assistant asks one short question with three defaults: the voice (two to four real voices of the text's language, by the names the computer uses; default the best female voice), the speed (default normal) and the place and format (default mp3 next to the text file, or in Downloads). «ok» takes the defaults; a request that already names them starts at once. It can keep the answers in `~/.config/text-to-speech.txt` if the person agrees.

## What this skill does on the computer

- Runs the speech program of the system (`tts`, `say`, Piper, espeak-ng or Windows PowerShell with System.Speech), and ffmpeg or ffprobe to make and check mp3 files.
- Reads the text the person points to; writes a prepared copy in a temporary folder and the audio file where the person said. Never changes the person's text file, never overwrites a file (a taken name gets « (2)»).
- Works offline. Connects to the internet only to install a tool or download a Piper voice, each after the person's yes.
- Plays nothing aloud unless asked.
- On Windows it runs a short PowerShell script it writes to the temporary folder, with `-ExecutionPolicy Bypass` for that one run.
- Stores nothing about the person, except `~/.config/text-to-speech.txt` with their voice, speed and folder, only if they agree.
- The installers are other people's programs with their own rules: `npx skills` (Vercel) sends an anonymous install count unless `DISABLE_TELEMETRY=1` is set.

## Tested on

2026-11-16, in a fresh Claude Code configuration (its own login, no user instructions, only this skill and two others of the same author (YouTube downloads, audiobooks), so that a near miss could start the wrong one): Claude Haiku 5.5 (`claude-haiku-5-5`) in Claude Code 2.1.325 on macOS 27.2 (Apple M3 Pro), `tts` from TTS 0.5, ffmpeg 9.0.2. Checked by script:

- «Озвучь текст из ru.txt» (Russian, no voice, speed or place named): the first reply was only the three questions; after «ok, go with the defaults», an mp3 of 19.4 s.
- «Read en.txt aloud into an mp3 in this folder, any voice, normal speed»: no question, an mp3 of 8.4 s.
- The skill started by itself for both requests, and for none of three near misses (an mp3 to m4a, a transcription of a recording, the sound out of an mp4).
- The same Russian request without the skill: an .aiff file instead of an mp3; on the way `say` was also run without an output file and read the text aloud through the speakers.

2026-11-17, Linux: Ubuntu 24.04 (arm64, a virtual machine), Piper 1.8.0 in its own virtual environment, ffmpeg 6.1.1, espeak-ng. The «Linux» commands of `SKILL.md`, run by hand: two voices downloaded (`ru_RU-irina-medium`, `en_US-lessac-medium`); a Russian text prepared for the ear, an mp3 of 9.4 s; an English text at 1.5× (`--length-scale 0.67`), an mp3 of 4.5 s; the same Russian text with espeak-ng. Read back by speech recognition: every number and the currency in place. The run found that Piper installed with `pipx` cannot download voices; the install section now uses a virtual environment.

2026-11-15, the first draft: the same model, as a sub-agent inside a configured Claude Code session (not a fresh configuration). Results checked by script (length, codec, streams) and read back by Whisper large-v3 and GigaAM:

- A Russian text with dates, times, money in dollars and hryvnias, percentages, gigabytes, a Roman century and «ИИ», with no voice, speed or place named: the first reply was only the three questions, listing the Mac's Russian Siri voices («Voice 2» female, «Voice 1» male) and Milena (Enhanced) with the defaults. After «male, 1.5 times faster, save in …»: an mp3 with Siri «Voice 1» at 1.5× (24 s against 36 s at normal speed), every number and sign read as words.
- An English text with everything named: no question, an mp3 with Siri «Voice 2», 18 s.
- The Russian text on a Mac without `tts`: the assistant wrote the numbers, dates, money and abbreviations as words in the right case itself and made the mp3 with `say` (Katya Enhanced); read back with every sum, unit and «искусственный интеллект» in place.
- The same Russian request without the skill: `say` with the compact Milena voice on the raw text; the hryvnia sign and «ИИ» were dropped and «ГБ» was spelled as letters.

Not tested: other models, other agents, Windows (its commands are from Microsoft's documentation), a model following the Linux commands (they were run by hand), the installs marked **docs**.

## What it cannot do

- Run in a chat app without a sandbox that can install Piper.
- Use the Siri voices without `tts`, or on Linux and Windows.
- Use Piper on a Mac: the pip build of Piper 1.6.1 to 1.8.0 for Apple Silicon writes empty files ([Piper issue 272](https://github.com/OHF-Voice/piper1-gpl/issues/272), open). The Mac's own voices are better there anyway.
- Clone a voice, change emotions or pronounce stress marks: it reads plain text with the voices the system has.

## Voices and the law

The macOS licence (section 2F of the macOS 27 Software License Agreement) allows the system voices, Siri voices included, for personal, non-commercial use and forbids publishing recordings of them. Files for yourself are fine; a video, a podcast or a product needs another voice. Piper voices carry their own licences, in the `MODEL_CARD` of each voice. The skill says this once when the person mentions publishing.

## License

MIT

# KnowIT

> **Built from curiosity, refined through experimentation.**

🇬🇧 English · [🇫🇷 Français](README.fr.md)

KnowIT is a growing collection of technical guides on computing topics, written in English and French.

It was born from one simple habit:

> **If I had to spend hours figuring something out, I'll make sure the next person doesn't have to.**

Every guide has one objective: to make a topic easier to understand than it was the first time I met it. The goal isn't to replace official documentation. It's to bridge the gap between the reference and real understanding, by explaining not only **what** something is, but **why** it works the way it does.

Think of it less as a textbook and more as the notes you'd wish someone had handed you before you started.

---

## Guides

| Topic | What you'll find | 🇬🇧 English | 🇫🇷 Français |
|---|---|---|---|
| **SQL** | From your first `SELECT` to joins, window functions, indexes and the classic mistakes | [Read](en/SQL_cheatsheet.md) | [Lire](fr/SQL_cheatsheet.md) |
| **Markdown** | A complete formatting cheat sheet, including GitHub Flavored Markdown | [Read](en/md_cheatsheet.md) | [Lire](fr/md_cheatsheet.md) |
| **The Windows `.exe` format** | The PE file structure from the DOS header to the data directories, with a plain-English ramp for beginners | [Read](en/The%20Windows%20Executable%20%28.exe%29%20Format.md) | [Lire](fr/The%20Windows%20Executable%20%28.exe%29%20Format.md) |
| **Samba 4 Active Directory** | Building an AD domain controller and joining an Ubuntu client with Samba, Kerberos, realmd and SSSD | [Read](en/Samba_setup.md) | [Lire](fr/Samba_setup.md) |
| **C & C++** | From the compilation pipeline and pointers to RAII, templates, undefined behavior and debugging, for beginners and veterans alike | [Read](en/C_and_Cpp_guide.md) | [Lire](fr/C_and_Cpp_guide.md) |

---

## Who is KnowIT for?

Anyone with an interest in learning. Whether you're discovering a topic for the first time, revisiting one you haven't touched in years, or just looking for a quick reminder, KnowIT is designed to help you understand, not just memorize.

- **Beginners** learning a concept from the ground up.
- **Students** who want explanations that prioritize understanding.
- **Professionals** who occasionally need a refresher.
- **Curious minds** who simply enjoy learning how things work.

---

## How is it organized?

KnowIT follows a simple principle:

> **One topic, one document.**

Each document focuses on a single subject and opens with its own table of contents. Concepts are introduced progressively, so a guide reads well front to back and still works as a quick reference later on.

Each language lives in its own folder, and **a guide keeps the same filename in every language**, so switching language is just a matter of swapping `en` for `fr` in the path.

```
KnowIT/
├── README.md              ← you are here
├── README.fr.md           ← version française
├── en/                    ← English guides
│   ├── C_and_Cpp_guide.md
│   ├── SQL_cheatsheet.md
│   ├── md_cheatsheet.md
│   ├── Samba_setup.md
│   └── The Windows Executable (.exe) Format.md
└── fr/                    ← guides en français
    ├── C_and_Cpp_guide.md
    ├── SQL_cheatsheet.md
    ├── md_cheatsheet.md
    ├── Samba_setup.md
    └── The Windows Executable (.exe) Format.md
```

---

## Contributing & requests

KnowIT is an evolving project. If you'd like to suggest a topic, improve an explanation, report an error, or contribute a translation, feel free to [open an issue](../../issues) or submit a pull request. Every contribution that makes a topic clearer, more accurate, or more accessible is welcome.

**Adding a translation?** Copy the guide into the other language's folder under the same filename, and keep the structure parallel: same sections in the same order, code left untouched, only comments and prose translated.

**Adding a new language?** Create a folder named after its two-letter code (for example `es/`), and add a column to the table above.

---

## The philosophy behind KnowIT

Most of the guides here weren't written because I already understood the subject. They were written because I wanted to understand it.

For me, writing is part of learning. It forces me to ask questions, challenge assumptions, connect concepts, and discover the parts I don't yet understand. By the time a guide is finished, I've usually learned as much from writing it as from reading about the topic.

That process shapes every document: each guide follows the path from curiosity to understanding, so whoever comes next can start a little further along it.

> **Writing is how I test my own understanding. If I can't explain a concept clearly, then I probably don't understand it well enough yet.**

# Agent Skills

Kumpulan skill untuk AI coding agent. Setiap skill mengikuti format
[Agent Skills](https://agentskills.io/) dan berada di dalam `skills/`.

## Skill tersedia

| Skill | Kegunaan |
| --- | --- |
| `filament-reviewer` | Review dan audit kode Filament, Laravel, keamanan, dan performa. |
| `git-push` | Analisis staged changes dan susun commit message Bahasa Indonesia. |
| `fitur-planner` | Generate dokumen perencanaan fitur baru yang aligned dengan PRD. |
| `vue3-reviewer` | Review dan refactor Vue 3, terutama Laravel + Inertia. |

## Instalasi

Lihat skill yang tersedia:

```bash
npx skills add crystrk/agent-skill --list
```

Instal seluruh koleksi:

```bash
npx skills add crystrk/agent-skill --all
```

Instal satu skill:

```bash
npx skills add crystrk/agent-skill --skill vue3-reviewer
```

Instal secara global (tersedia di semua project):

```bash
npx skills add crystrk/agent-skill -a -g opencode
```

Atau untuk agent tertentu:

```bash
npx skills add crystrk/agent-skill --skill vue3-reviewer -a -g opencode
```

## Struktur

```text
skills/
├── filament-reviewer/
│   ├── SKILL.md
│   └── references/
├── fitur-planner/
│   └── SKILL.md
├── git-push/
│   └── SKILL.md
└── vue3-reviewer/
    ├── SKILL.md
    └── references/
```

Setiap folder skill wajib memiliki `SKILL.md` dengan frontmatter `name` dan
`description`. Materi rinci ditempatkan di `references/`, sedangkan otomasi
opsional ditempatkan di `scripts/` agar `SKILL.md` tetap ringkas.

## Community

Skill ini juga terdaftar di [skills.sh](https://skills.sh/crystrk/agent-skill) —
direktori resmi ekosistem Agent Skills oleh Vercel. Lihat statistik pemasangan
dan trending skill di dashboard tersebut.

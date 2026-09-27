# docs

Kumpulan dokumentasi dan catatan perencanaan (plans & specs) lintas project dalam format Markdown.

## Cara Setup

Folder ini hanya berisi dokumen Markdown, bukan aplikasi — tidak ada yang perlu di-install atau di-build.

### Membaca langsung
Buka file `.md` di dalam `superpowers/plans/` dan `superpowers/specs/` dengan editor apa pun (VS Code, Obsidian, dsb).

- **VS Code**: buka folder ini lalu tekan `Cmd/Ctrl + Shift + V` untuk preview Markdown.

### Menyajikan sebagai situs statis (opsional)
Jika ingin membaca sebagai website:

```bash
# Opsi MkDocs
pip install mkdocs
mkdocs serve

# Opsi Docsify
npm i -g docsify-cli
docsify serve .
```

## Struktur

- `superpowers/plans/` — rencana implementasi per fitur/fase.
- `superpowers/specs/` — dokumen desain/spesifikasi.

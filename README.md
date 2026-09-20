# mime

[![CI](https://github.com/alya-lang/mime/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/mime/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/mime?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fmime%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fmime%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

MIME type and media type detection library for Alya.

---

## 🌟 Features

- ⚡ **High Throughput**: >3.4M lookups/sec with zero heap allocations for lookup queries.
- 🌐 **Comprehensive Registry**: 100+ standard MIME types covering web markup, stylesheets, scripts, documents, audio, video, images, fonts, archives, and programming languages.
- 🎯 **Smart Path & URL Resolver**: Automatically extracts extensions from Unix paths (`/var/www/index.html`), Windows paths (`C:\app\index.js`), URLs with query strings and anchors (`style.css?v=2#dark`), and multi-dot filenames (`archive.tar.gz`).
- 🔠 **Full Content-Type Formatter**: Emits ready-to-use HTTP `Content-Type` headers with standard charset parameters (e.g. `text/html; charset=utf-8`).
- 🔄 **Bidirectional Lookups**: Fast extension-to-MIME forward lookup and MIME-to-extension reverse lookup.
- 🔍 **Media Classification**: Instant `is_text` and `is_binary` inspection for HTTP gzip/brotli compression gating or file upload verification.
- 🧩 **RFC Header Parser**: Parses complex `Content-Type` strings (with parameters and custom charsets) into structured `MimeType` records.

---

## 📁 Project Architecture

```text
mime/
├── alya.toml               # Package manifest
├── src/
│   ├── lib.alya            # Public API facade
│   ├── types.alya          # MimeType struct, extract_ext, path utilities
│   └── core/
│       ├── db.alya         # Extension <-> MIME bidirectional lookup tables
│       ├── charset.alya    # Default charsets & media classification rules
│       └── parser.alya     # Content-Type header parser
├── examples/
│   └── demo.alya           # HTTP file server dispatcher example
├── tests/
│   └── test_basic.alya     # Automated test suite (73 assertions)
└── benches/
    └── bench_basic.alya    # Micro-benchmarks
```

---

## 📦 Installation

Add `mime` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
mime = { git = "https://github.com/alya-lang/mime", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alya add mime --git https://github.com/alya-lang/mime --branch main
alya install
```

---

## 🚀 Quick Start

```alya
import "mime"

function main()
    # 1. Look up MIME essence
    say mime::lookup("index.html")                    # "text/html"
    say mime::lookup("image.PNG")                     # "image/png"
    say mime::lookup("/var/www/app.js?v=2")           # "application/javascript"

    # 2. Get full Content-Type header for HTTP servers
    say mime::content_type("style.css")               # "text/css; charset=utf-8"
    say mime::content_type("hero.png")                # "image/png"
    say mime::content_type("unknown.xyz")             # "application/octet-stream"

    # 3. Reverse lookup (MIME to canonical extension)
    say mime::extension("application/json")           # "json"
    say mime::extension("image/svg+xml")              # "svg"

    # 4. Media inspection for compression or routing
    if mime::is_text("/public/bundle.js")
        say "Eligible for HTTP compression (gzip/brotli)"
    end

    # 5. Parse Content-Type header
    let parsed = mime::parse("text/html; charset=utf-8")
    say "Type:    " + parsed.type_name                # "text"
    say "Subtype: " + parsed.subtype                  # "html"
    say "Charset: " + parsed.charset                  # "utf-8"
end

main()
```

---

## 📖 API Reference

### Core Functions

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `lookup(path_or_ext, default_type)` | `path_or_ext: string, default_type: string = "application/octet-stream"` | `string` | Looks up MIME essence (e.g. `"text/html"`). Falls back to `default_type` if unknown. |
| `type_by_ext(ext)` | `ext: string` | `string` | Returns raw MIME essence for extension without fallback (`""` if not found). |
| `content_type(path_or_ext, default_type)` | `path_or_ext: string, default_type: string = "application/octet-stream"` | `string` | Returns full `Content-Type` header with charset if applicable (e.g. `"text/html; charset=utf-8"`). |
| `extension(mime_type)` | `mime_type: string` | `string` | Reverse lookup: returns default file extension (e.g. `"json"` for `"application/json"`). |
| `is_text(path_or_mime)` | `path_or_mime: string` | `int (1 or 0)` | Returns `1` if media type is text-based (e.g. `text/*`, `application/json`, `image/svg+xml`). |
| `is_binary(path_or_mime)` | `path_or_mime: string` | `int (1 or 0)` | Returns `1` if media type is binary data. |
| `charset(path_or_mime)` | `path_or_mime: string` | `string` | Returns default charset (`"utf-8"` for text types, `""` for binary). |
| `with_charset(mime_essence, charset_name)` | `mime_essence: string, charset_name: string = "utf-8"` | `string` | Appends charset parameter to a MIME essence. |
| `parse(header_str)` | `header_str: string` | `MimeType` | Parses a `Content-Type` header into a structured `MimeType` instance. |
| `format(mime_obj)` | `mime_obj: MimeType` | `string` | Formats a `MimeType` struct back into a `Content-Type` string. |

### Data Structures & Enums

#### `MimeCategory`

```alya
pub enum MimeCategory
    Text = 1,
    Image = 2,
    Audio = 3,
    Video = 4,
    Application = 5,
    Font = 6,
    Model = 7,
    Multipart = 8,
    Other = 9
end
```

#### `MimeType`

```alya
pub struct MimeType
    essence      # "text/html"
    type_name    # "text"
    subtype      # "html"
    charset      # "utf-8" or ""
    is_text      # 1 or 0
    is_binary    # 1 or 0
end

# Struct Methods:
# - parsed.is_text() -> 1 or 0
# - parsed.is_binary() -> 1 or 0
# - parsed.to_string() -> formatted header string (e.g. "text/html; charset=utf-8")
```

---

## 🧪 Running Tests & Benchmarks

Run the automated test suite using `alya test`:

```bash
alya test
```

Generate static API documentation:

```bash
alya doc . -o docs --markdown
```

Run the benchmark suite:

```bash
alya run benches/bench_basic.alya
```

Run the realistic HTTP file server dispatcher demo:

```bash
alya run examples/demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   alya fmt . --check
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
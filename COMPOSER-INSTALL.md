# Installing grs-standards via Composer

Goal: keep Copilot instructions and coding standards in sync across projects using a single shared package.

Do this once in the **consuming** project’s `composer.json`:

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/giantrocketship/grs-standards.git"
        }
    ],
    "require-dev": {
        "giantrocketship/grs-standards": "dev-main"
    },
    "scripts": {
        "post-install-cmd": [
            "mkdir -p .github",
            "cp vendor/giantrocketship/grs-standards/copilot-instructions.md .github/copilot-instructions.md || true",
            "cp vendor/giantrocketship/grs-standards/CODING-STANDARDS.md ./CODING-STANDARDS.md || true"
        ],
        "post-update-cmd": [
            "@post-install-cmd"
        ]
    }
}
```

Notes:

- You can merge this into an existing `composer.json` (for example, by adding just the `giantrocketship/grs-standards` entry to your existing `require-dev`, and appending the `scripts` entries).
- The `|| true` ensures the script doesn’t fail the install if the files are missing for some reason.

After this is saved, simply run your normal Composer commands (`composer install` / `composer update`). Composer will:

1. Download `giantrocketship/grs-standards` into `vendor/`.
2. Copy the latest `copilot-instructions.md` into `.github/copilot-instructions.md` (where GitHub Copilot reads project instructions).
3. Copy the latest `CODING-STANDARDS.md` into your repo root.

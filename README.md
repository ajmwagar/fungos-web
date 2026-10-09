# fungOS website

Published static website in `dist/`; hosting uses `netlify.toml`.

Shared layout and the project switcher are generated from
`ajmwagar/fpl-project-theme`. `dist/theme/REVISION` pins the exact source revision.
`dist/distro.css` imports that bundle and contains only fungOS-specific styles.
No theme server, package install or private Git clone is needed at deploy time.

To refresh from the theme checkout:

```sh
cargo run -- --site ../fungos-web/dist --project fungos
cargo run -- --site ../fungos-web/dist --project fungos --check
```

The source website snapshot is also produced by the distribution tooling.
After replacing `dist/` from that source, run the shared theme installer again
before committing/publishing; otherwise the shared component can be lost.

Theme CSS is AGPL-3.0-or-later; its full license ships at `dist/theme/LICENSE`.
This does not relicense site content or upstream artwork. Asset terms are in
`ASSET-NOTICES.md` and `dist/theme/ASSET-NOTICES.md`.

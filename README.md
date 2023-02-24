# Pandoc patch for obsidiansystems/obelisk

This is a patch to [Pandoc](https://github.com/jgm/pandoc) for use as Haskell
library with [Obsidiansystems Obelisk](https://github.com/obsidiansystems/obelisk).

This isn't tested by any means, but seems to be working fine.

## What this patch does

The `skylighting` library is fixed to the very old version 0.9.
This is necessary because new versions introduce a C-dependency via `xml-conduit`.

`aeson` is downgraded to version 1.5.6.0 with hopefully no big consequences.

The writer for ConTeXt is completely removed.
It depends on features of the new skylighting versions.

## How to use

See [learn-palantype](https://github.com/rubenmoor/learn-palantype) for a project
that uses Pandoc with this patch, especially check the
[default.nix](https://github.com/rubenmoor/learn-palantype/blob/main/default.nix).
In `default.nix` all versions that needs pinning are pinned.

## Why is a patch necessary

The obelisk framework allows to build web applications (and smartphone apps)
using Haskell only, implementing [reflex-frp](https://github.com/reflex-frp/reflex)
for interactive UIs.

Obelisk projects are split into three cabal packages: `frontend`, `common`, and `backend`.
The `frontend` and `common` packages are compiled by both, GHC and GHCJS.
The latter one is necessary to have Haskell programs running inside browsers.

Obelisk is also stuck with GHC 8.
The latest release uses GHC 8.6.4, the development branch uses GHC 8.10.7.

### Using Pandoc with Obelisk

The use of any library with an obelisk project needs to adapt to these compilers.
`backend` dependencies need to work with the older GHC version, which usually
isn't a big deal.
Dependencies to `frontend` and `common` additionally can't depend on C compilers.
And those dependencies on C sometimes sneak in and ruin your day.

For `reflex-frp`, there is [reflex-dom-pandoc](https://github.com/srid/reflex-dom-pandoc),
which allows you to use Pandoc's document representation
[Pandoc](https://hackage.haskell.org/package/pandoc-types-1.23/docs/Text-Pandoc-Definition.html#t:Pandoc)
as input to a reflex component `elPandoc`,
effectively rendering `Pandoc` to HTML.

`reflex-dom-pandoc` has a dependency to `pandoc-types` (not Pandoc), which is great:
Having `frontend` or `common` depend on the whole `Pandoc` package would cause all kinds of problems.
Instead, I opt to leave the Pandoc-dependency with `backend` (and thus any
document conversions happend on the server), while the frontend only renders the
`Pandoc` type.

### The technical problems

There is a choice to be made between Pandoc versions.
The newest version not only has the latest features, but also, all dependencies
to Lua have been separated and are not part of Pandoc the haskell library.

An older Pandoc version would potentially work out of the box with one of the
GHC versions that are demanded by obelisk
(8.6.4 or 8.10.7), but I opted to make Pandoc 3.1 work with GHC 8.10.7 as it
turned out not to be a big deal.

For `pandoc-types`, there is a C-dependency, unfortunately, that we need to get
rid of.

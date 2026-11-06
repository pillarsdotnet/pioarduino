# pioarduino

The PlatformIO community fork packaged for Ubuntu from the upstream PyPI
sdist, published to the
[Forgejo Debian registry](https://forgejo.bobbalisa.org/pillarsdotnet/-/packages)
for **noble** (24.04), **resolute** (26.04) and **stonking** (26.10).

pioarduino keeps up with current Espressif platforms — ESP32-C6, ESP32-H2,
ESP32-P4 and the Arduino-ESP32 3.x cores — which upstream PlatformIO 6.2 does
not carry. It is in no Ubuntu archive and never has been.

**It conflicts with `platformio`.** The distribution is named `pioarduino`,
but the module inside it is still `platformio`, two of the three entry points
are `platformio` and `pio`, and the udev rule has the same filename — so the
deb shares every path with
[`platformio`](https://forgejo.bobbalisa.org/pillarsdotnet/platformio) and
carries `Conflicts`, `Replaces` and `Provides` on it. Install one or the
other; `Provides` means anything depending on `platformio` accepts this.

## What is in here, and what is not

**Only `debian/` and the CI.** No copy of the source and no orig tarball:
the sdist is fetched from PyPI at build time and checked against the sha256
in `debian/upstream/sdist.sha256`. The layout, the build and the version
scheme are `platformio`'s, and that repository's README describes them in
full.

```
debian/
    control  rules  install  docs  copyright  changelog
    patches/relax-deps.patch, series
    upstream/sdist.sha256     # the pin: name and sha256 of the sdist
    tests/control             # autopkgtest smoke tests
    README.source             # how a version bump works
.forgejo/
    workflows/registry.yml    # the three-series matrix
    workflows/mirror.yml      # rebuild the github branch, push it to GitHub
    registry-upload           # the whole build
```

## The one patch

`relax-deps.patch` widens two pins. Neither changes what the fork does, and
neither changes the binary package's dependencies — `dh_python3` discards
PEP 440 constraints when it generates `${python3:Depends}`, measured both
ways on `platformio`, which uses this same packaging. What they change is the
requirement metadata the installed package records in `Requires-Dist`, where
upstream's pins would have it declare requirements the system it is installed
on cannot satisfy.

| Pin | Becomes | Why |
| --- | --- | --- |
| `bottle == 0.13.*` | `bottle >= 0.12` | noble ships 0.12.25, and the only use of bottle is one `bottle.template()` call, the same in 0.12 and 0.13 |
| `starlette < 1` | `starlette < 2` | stonking ships 1.7.0, so the fork's own pin excludes the only version that series has |

The `starlette` widening is safe because the fork's starlette-facing code is
*identical* to upstream PlatformIO 6.2.0 — the same seven imports from
`starlette.applications`, `.middleware`, `.responses`, `.routing`,
`.staticfiles`, `.status` and `.endpoints` — and upstream declares `< 2` and
is built against stonking's 1.7.0 in this same registry.

Upstream's `click >=8.0.4, <8.4` is left alone: stonking's 8.3.3 fits it.

## Why 6.2.0 and not 6.2.0.post1

`6.2.0.post1` is the current PyPI release and is **deliberately not taken**.
It replaces the `starlette < 1` pin with `starlette >=1.3.1, <2`:

```
noble      starlette 0.31.1    too old
resolute   starlette 0.50.0    too old
stonking   starlette 1.7.0     fine
```

Upstream raised that floor on purpose, so taking `post1` means either
dropping noble and resolute, or overriding a floor upstream set with
knowledge we do not have. 6.2.0's `< 1` pin is the opposite problem — a
ceiling that is demonstrably wrong, since the code is upstream PlatformIO's
and runs against 1.7.0 — which is why widening *that* is safe and lowering
the other would not be.

Revisit when noble and resolute are no longer targets, or when the fork's
starlette-facing code is checked against 1.3 and found to need it.

## Versions

```
6.2.0-1~ppa2~ubuntu24.04.1      noble
6.2.0-1~ppa2~ubuntu26.04.1      resolute
6.2.0-1~ppa2~ubuntu26.10.1      stonking
```

`~ppa1` was built by hand in October 2026 from a packaging tree that no
longer exists; `~ppa2` is the first revision any checkout can reproduce, and
outranks it.

## Related

- [`platformio`](https://forgejo.bobbalisa.org/pillarsdotnet/platformio) —
  upstream PlatformIO, packaged the same way, and the fuller write-up of how
  that packaging works.
- [`python-ajsonrpc`](https://forgejo.bobbalisa.org/pillarsdotnet/python-ajsonrpc)
  — the backport this needs on noble and resolute.

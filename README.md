# GeckoLib 5.5.2 — Float Precision Fix

An **unofficial patched build of GeckoLib 5.5.2 for Fabric / Minecraft 26.1.2**.

This build backports the fix proposed in upstream GeckoLib PR #884:

- Upstream: https://github.com/bernie-g/geckolib/pull/884
- GeckoLib: https://github.com/bernie-g/geckolib
- Minecraft target: **26.1.2**
- Loader: **Fabric**
- Base GeckoLib version: **5.5.2**

## Why this patch exists

On long-running worlds, GeckoLib's `ClientUtil.getCurrentTick()` can lose time precision because `ClientLevel.getGameTime()` is converted from `long` to `float` before the partial tick is added.

Conceptually, the affected calculation is:

```java
(float) mc.level.getGameTime() + partialTick
```

This patched build changes the calculation to perform the addition in `double` precision:

```java
(double) mc.level.getGameTime() + (double) partialTick
```

The public method signature is **not changed**. `getCurrentTick(Float)` continues to return `double`.

## Confirmed downstream reproduction

The issue was independently reproduced with:

- **Minecraft:** 26.1.2
- **Fabric**
- **GeckoLib:** 5.5.2
- **Vic's Point Blank:** 2.2.0

In the affected survival world, the world time was approximately:

```text
16,850,073 ticks
```

The weapon animation was visibly much lower-FPS than in a new world while the overall Minecraft FPS remained normal.

As an additional diagnostic test, changing only the world's `level.dat` `Time` value to a much smaller number immediately restored smooth weapon animation.

After applying this GeckoLib patch, the original world time could be kept while the Point Blank weapon animation returned to normal.

This behavior is consistent with the long-uptime float precision issue discussed in upstream PR #884.

## Scope of the patch

Only this GeckoLib class is modified:

```text
com/geckolib/util/ClientUtil.class
```

The patched archive contains the same GeckoLib mod identity/version metadata as GeckoLib 5.5.2.

The public API signature remains:

```text
getCurrentTick() -> double
getCurrentTick(Float) -> double
```

The patch changes the internal numeric conversion/operation from a `float`-based calculation to a `double`-based calculation.

## Installation

1. Make a backup of your Minecraft instance and worlds.
2. Remove the official GeckoLib 5.5.2 JAR from the `mods` folder.
3. Put this JAR in the same `mods` folder.
4. Do **not** keep both GeckoLib JARs installed at the same time.
5. Start Minecraft 26.1.2 with Fabric.

Recommended file name:

```text
geckolib-fabric-26.1.2-5.5.2-float-precision-fix.jar
```

## Important compatibility note

This build intentionally keeps GeckoLib's public method signature unchanged. Normal mods calling:

```java
ClientUtil.getCurrentTick(...)
```

should continue to link against the same method descriptor.

However, this is still an **unofficial patched binary**. Mods or tools that depend on GeckoLib's exact internal bytecode layout may require separate verification.

## Verification

SHA-256:

```text
124ba3322de858a6b5b2d7e0cb912b920503352c679f494093caddae6b8f9212
```

## Upstream status

This project does **not** claim authorship of the original GeckoLib bug fix.

The upstream fix was proposed by **ImBit** in GeckoLib PR #884:

https://github.com/bernie-g/geckolib/pull/884

This repository provides an independently tested backport for the **GeckoLib 5.5.2 / Minecraft 26.1.2 Fabric** combination.

## Related Point Blank report

A related Point Blank issue reports inconsistent weapon animation smoothness between worlds despite similar overall FPS:

https://github.com/vicmods/pointblank-issues/issues/32

The current project adds a reproducible downstream case connecting that symptom to GeckoLib's long-uptime time precision problem.

## Known limitations

This patch addresses the `ClientUtil.getCurrentTick()` float precision problem described above.

It does **not** claim to fix every Point Blank rendering/performance issue. In particular, a separate issue where opening the Weapons Printer can occasionally cause the entire game to drop to very low FPS was observed during testing and should be treated as a separate problem until independently confirmed.

## Credits and license

- **GeckoLib:** Gecko / GeckoLib contributors
- **Original upstream fix proposal:** ImBit, PR #884
- **Downstream reproduction:** Vic's Point Blank 2.2.0

GeckoLib is distributed under the **MIT License**. This repository should retain the original GeckoLib copyright and license notices when redistributing modified GeckoLib code/binaries.

See the GeckoLib project for the authoritative license:

https://github.com/bernie-g/geckolib/blob/main/LICENSE

## Disclaimer

This is an unofficial patched build. It is not an official GeckoLib release.

Use backups when replacing libraries in an existing Minecraft instance.

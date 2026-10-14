# SilentPatch for Bully: Scholarship Edition — Community FIX

This game, which shares a lot of the internals with GTA games, performs fairly well in its PC incarnation as is.
However, it's more than likely that you have at some point spotted the amount of complaints Windows 10
users have about the game, or maybe you have encountered crashes yourself.

SilentPatch attempts to fix Bully memory management completely, so it behaves in the same way independent
of Windows version. This is not the only fix included, however - most notably, it attempts to improve
gameplay experience by improving frame pacing, as well as fixing a few other issues.

The **Community FIX** branch extends the original SilentPatch beta with a borderless windowed mode (the stock
game only runs in exclusive fullscreen), plus a few build-system fixes so the project compiles on modern
Visual Studio toolchains out of the box.

A prebuilt release lives in [`Release-CommunityFIX/`](./Release-CommunityFIX) — grab `SilentPatchBully.asi`
and `SilentPatchBully.ini`, drop them next to `Bully.exe` with any ASI loader (e.g. Ultimate ASI Loader as
`dinput8.dll`), and you're done. Full instructions are in [`Release-CommunityFIX/INSTALL.txt`](./Release-CommunityFIX/INSTALL.txt).

Fixes featured in this plugin:

### Crash and bug fixes
* Collision loading code has been improved, fixing occasional crashes on initial game load
* Fixed game's objects pool usage, fixing possible crashes
* Fixed an occasional crash when starting Nutcrackin' or Music Class
* Fixed numerous instances of memory corruption on game exit
* Fixed an use-after-free in sound streaming code, causing a rare crash when talking to people
* Fixed handle leaks in audio code, preventing handles from accumulating during the game
* Fixed several memory leaks in audio code, preventing out of memory crashes during extended play sessions
* Made memory manager workarounds toggleable via the INI file - disabled by default, to be removed in the future
* Frame Limiter has been made much more precise, so the game should lock at exactly 30FPS now
  (as opposed to stock limiter being prone to dropping frames a lot)
* Fixed an issue where game would use more CPU than required when minimized

### Quality of life improvements
* An option to change FPS cap has been added to SilentPatchBully.ini file (game defaults to 30FPS)
* **FILE_FLAG_NO_BUFFERING** flag has been removed from IMG reading functions - potentially speeding up streaming

### Community FIX additions
* **Borderless windowed mode** — toggle with `BorderlessWindowed=1` in `SilentPatchBully.ini`. Forces the
  game to run inside a borderless popup window with D3D9 in windowed mode instead of exclusive fullscreen,
  so you can alt-tab instantly, use multiple monitors, overlays, streaming apps, etc. Implemented via IAT
  hooks on `CreateWindowExA`, `ChangeDisplaySettingsA`, and a dynamic intercept of `GetProcAddress` to
  wrap `Direct3DCreate9` + vtable-patch `IDirect3D9::CreateDevice` and `IDirect3DDevice9::Reset`
  (Bully resolves D3D9 at runtime, so a static IAT hook on `d3d9.dll` won't catch it).
* Optional `WindowWidth` / `WindowHeight` keys to force a specific window size (0 = use the game's own
  resolution setting, which is the recommended value).
* `Reset_Hook` hardening — the device is kept alive when the window is minimized and 0×0 backbuffer
  dimensions are substituted with the real client rect, preventing the "crash after a few alt-tabs"
  pattern that showed up in windowed D3D9 recovery paths.

## Compilation requirements

The original project was authored against Visual Studio 2017 with the `v141_xp` toolset and the Windows
7.0 SDK. The Community FIX branch is build-tested on Visual Studio 2022 (`v143` toolset, Windows 10 SDK)
without touching the `.vcxproj` — pass the overrides on the MSBuild command line:

```
msbuild SilentPatchBully\SilentPatchBully.vcxproj ^
  /p:Configuration=Release /p:Platform=Win32 ^
  /p:PlatformToolset=v143 /p:WindowsTargetPlatformVersion=10.0
```

Two tiny source-level build fixes are included so the modern SDK accepts the code:

* `WINVER` / `_WIN32_WINNT` bumped from `0x0502` (XP) to `0x0600` (Vista) — the pre-existing
  `CancelIoEx` call in the sound-stream fix already required Vista; this just makes it visible.
* `_SILENCE_STDEXT_ARR_ITERS_DEPRECATION_WARNING` defined to silence the deprecation of
  `stdext::make_checked_array_iterator` in `MemoryMgr.h`, which modern MSVC promotes to an error.

The `Utils/` submodule (CookiePLMonster/ModUtils) is pinned to commit **`1b81f57`** — later commits
rewrote the API (`PATCH_JUMP` → `HookType::Jump`, etc.) and are incompatible with the mod's source.

## Submitting feedback

Since this is a public beta release, you may encounter crashes. Because of this, MiniDumper utility has
been shipped together with SilentPatch. In case of a crash, a .dmp file will be created in your game directory.

If you want to report it as a bug (any feedback is very much appreciated), first **ENSURE YOU HAVE AN UNMODDED GAME**
(texture mods are fine, scripts - not so much). You can report a bug (.dmp file + a brief explanation on what
you were doing when the game crashes) in the Issues page.

## Credits

* [P3ti](https://github.com/P3ti) - co-developer
* [amzy](https://www.twitch.tv/amzy) - testing, overall support
* **AmenHermano** - Community FIX branch: borderless windowed mode, modern-toolchain build fixes


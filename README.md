# electron-flatpak-boilerplate
> [!IMPORTANT]
> [Flathub's Electron boilerplate](https://github.com/flathub/org.flathub.electron-sample-app) has been adapted to support the latest version of Electron
> So this boilerplate is now just an alternative way to do the same.
> Thus, it will no longer be actively maintained, unfortunately.

This boilerplate (works offline) assumes you have the following:

- A working Electron app (obviously).
- And a vendored tarball; this should have your app's code (`main.js`, `preload.js`, etc), any licenses and in-app libraries for them, a README (if you want), any icons and any `.desktop` or `.metainfo.xml` files (for Flathub), and the Electron binaries you need (such as `electron-v40.0.0-linux-x64.zip` from [the official Electron GitHub repo](https://github.com/electron/electron/releases) in the `cache/` folder of your tarball (you can also add this in the `sources` property of the YAML as a file so it isn't extracted; this is because Electron build tools expect a zipped archive containing any needed binaries).

To know more about how to use this template, you can [read the YAML](https://github.com/A-Star100/electron-flatpak-boilerplate/blob/main/org.electronjs.flatpak_boilerplate.yml).

# FAQ
## My app keeps crashing due to some weird DBUS/MESA-related error
This *usually* means that your app is trying to launch using native Wayland when the version of Electron you're using doesn't support it.
To force XWayland (an X11-compatibility layer) for the best compatibility, add 
```yaml
- --socket=x11
```
(unless 
```yaml
- --socket=fallback-x11
```
is already there) to the `finish-args` of your YAML,
remove 
```yaml
- --socket=wayland
```
(if it is there) to avoid linter errors, and add the following to your `run.sh` script:
- ```yaml
  - 'export WAYLAND_DISPLAY=""'
  ```
 and
- ```yaml
  - 'export XDG_SESSION_TYPE="x11"'
  ```
 to your YAML to **force** XWayland no matter what.

If any more issues occur, as a last resort add the flag 
```yaml
--ozone-platform=x11
```
to your `run.sh`'s `zypak-wrapper` line.
For example:
```yaml
sources:
  - type: script
  dest-filename: run.sh # final filename
  commands:
    - 'export WAYLAND_DISPLAY=""' # manually unset WAYLAND_DISPLAY to force XWayland
    - 'export XDG_SESSION_TYPE="x11"' # also set session type for the sandbox to be x11 to further force XWayland
    - 'exec zypak-wrapper /app/lib/MyAwesomeApp/MyAwesomeApp --ozone-platform=x11 "$@"' # uses zypak to execute the compiled binary - zypak comes with the Electron BaseApp but if you add it as a source to be in
```
## My builds keeps failing when running `npm install` or due to ENOTCACHED (even if assets are cached)
This means that you're attempting to **get packages from INSIDE the build process**, which isn't great for [security reasons](https://github.com/flathub/flathub/issues/3392#issuecomment-1207174370), so it is blocked by default. 

Ideally, you would add a source inside `sources` so any needed files would be downloaded **BEFORE** the build process.
In order to fix this you will probably need to rewrite a lot of your YAML. The YAML in this repository doesn't have these errors because I rewrote it after realizing this.

If you **really want to allow network access (note that if you are publishing to Flathub the reviewers will ask for this to be removed from the manifest!)**,
then you can add:
```yaml
# add this to the build-options in your YAML
build-args:
  - --share=network
```
**but DON'T DO THIS this unless it's for local builds!**

## Why does my app show DBUS errors on launch but after a few seconds launches fine
This is because Electron attempts to access its DBUS before it is properly initialized.
You can ignore this, it doesn't affect any functionality, and doesn't matter at all.

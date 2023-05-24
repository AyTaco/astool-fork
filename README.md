# astool: All Stars Utility Toolkit
# forked for readme
# Original: https://github.com/kirara-research/astool

Requires Python 3.7+.

## Installation

It's recommended that you install using pip (`pip install .`) over using setup.py.
To enable the use of async for downloading packages (massive speedup over requests
if you have a high bandwidth internet connection), do `pip install '.[async_pkg]'`.

You also need to install the hwdecrypt extension module the same way
(`pip install ./hwdecrypt_src`), unless you're arposandra and include your own copy.

## Storage

You can set the `ASTOOL_STORAGE` environment variable to change where data gets
stored. The default is the current directory, so it's recommended to choose a
better place.

Files:
- [server]/astool_store.json - Contains the account credentials used by astool, as well
  as the last known master version.
- [server]/cache/pkg... - Encrypted asset packages. Assets are retrieved from them as needed.
- [server]/masters/.../... - Contains asset databases. Each master version has its own folder
  with its collection of databases.

## Command-Line Usage

### astool

All commands start with the common invocation:

```sh
python -m astool [common args] [server] [command args...]
```

Common args can include:

- `-b (bundle ver)` sets the app version, which affects DB encryption keys and server URLs.
- `-f (name)` sets the memo's filename. The actual path used will be `$ASTOOL_STORAGE/[server]/[name].json`.
- `-q` quiets some logs.

Commands and their arguments:

- `accept_tos` - Accepts the TOS, only needs to be used if not done by `bootstrap`. Currently not working for
  EN.
- `bootstrap` - Creates an account and saves the login info to the memo. Currently only partially working for EN.
- `current_master` - Prints the current master from the memo to stdout.
- `decrypt_master (file)` - Decrypts the database (file) to (file).dec. The directory that (file)
  is in will be consulted for the appropriate keys.
- `dl_master [-m version] [-f]` - Downloads the databases associated with version (version).
  If (version) is not given, will ask the API for the latest version. If -f is given, databases
  will be redownloaded even if they exist.
- `invalidate` - Removes fast resume data from the memo. This will force a relogin on the next API call.
- `master_gc` - Deletes decrypted databases that can be recreated. The latest master will not be deleted.
- `pkg_sync [flags] (groups...)` - Check the downloaded package cache and see if the named groups are
  complete. If not, and you haven't told it to validate only, it will download the missing parts `%` will download all.
  Flags:
  - `-m/--master [version]` - Use the specified master databases. If this is not given, it'll use the 
    current version from the astool memo.
  - `-sfd/--signal-cts [path]` - The string "ready\n" will be written to this path when all API calls
    are finished, before any packages are downloaded.
  - `-n/--validate-only` - Don't download anything. For `pkg_gc`, don't delete anything.
  - `-g/--lang [language]`: Use the specified language's asset database.
- `pkg_gc` - Deletes any packages files that are not referenced by the current asset database.
  Flags:
  - `-m/--master [version]` - Use the specified master databases. If this is not given, it'll use the 
    current version from the astool memo.
  - `-n/--dry-run` - Don't delete anything. Note that this is a different long option from pkg_sync.
  - `-g/--lang [language]`: Use the specified language's asset database.

### astool_extra (Data Extraction)

```sh
python -m astool_extra.unpack_fs [common args]
```
Common args can include:

- `-h (--help)` - Shows the help menu.
- `-r (--region)` - Tells astool what region to use (`en` or `jp`).
- `-m None, --master` - Master version (default: latest known via dl_master)
- `-l (--lang)` - Tells astool what language to use (default: default for server region).
- `-t (--table-list) `- Comma-separated list of tables to extract (default: all). Pass 'list' to see available.
- `-y (--skip-confirmation)` - Don't ask before extracting

`[output]`: Output folder

List of tables:

- `adv_script`
- `background`
- `gacha_performance`
- `live2d_sd_model`
- `live_prop_skeleton`
- `live_timeline`
- `m_asset_sound`
- `m_movie`
- `member_facial`
- `member_facial_animation`
- `member_model`
- `member_sd_model`
- `navi_motion`
- `navi_timeline`
- `shader`
- `skill_effect`
- `skill_timeline`
- `skill_wipe`
- `stage`
- `stage_effect`
- `texture`

## Programmatic Usage

Most APIs in the astool.pkg, astool.ctx, astool.masters, and astool.iceapi modules are available
for public use.

## Decryption

Assets are encrypted using a LCRNG-based stream cipher. See libpenguin/penguin_tool.c
for the algorithm. The RNG seeds are stored in the manifest (see karstool) or in
the asset database.

Audio files are stored as CRI HCA like every other game. The key for them is
`0x5a2f6f6f0192806d`, or `-a 0192806d -b 5a2f6f6f`.

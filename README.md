# Askone

Askone downloads the tracks of your Spotify playlists from YouTube, on Windows, from a desktop window or from the
`askone` command line. Paste a playlist link, list it, untick what you do not want, download: each track is found on
YouTube, saved as MP3 (or M4A, Opus, FLAC) and tagged with its title, artists, album and cover.

This repository only publishes the installers; the source code is not public.

> **Terms of use.** Askone is for personal use. YouTube's terms of service do not allow downloading its content, and
> the tracks you download may be protected by copyright. You are responsible for what you download and how you use
> it. Askone comes with no warranty.

- [Install](#install)
- [First run](#first-run)
- [When YouTube asks whether you are a robot](#when-youtube-asks-whether-you-are-a-robot)
- [Command line](#command-line)
- [Updating](#updating)
- [Uninstalling](#uninstalling)
- [Where Askone keeps its files](#where-askone-keeps-its-files)
- [When something goes wrong](#when-something-goes-wrong)

## Install

Download `Askone setup v<version>.msi` from the [Releases](https://github.com/PrometeoConseil/Askone-releases/releases)
page (about 50 MB) and run it. Nothing else has to be installed first: the .NET runtime is inside, and the tools
Askone drives (yt-dlp, Deno, ffmpeg) are downloaded on first use. Windows 10 or 11, 64-bit.

The installer asks whether to install for you alone or for every user of the computer:

| | Just for you (default) | For everyone |
| --- | --- | --- |
| Folder | `%LOCALAPPDATA%\Programs\Askone` | `C:\Program Files\Askone` |
| `askone` command | on your PATH | on the system PATH |
| Administrator rights | not needed | needed |

It adds Askone to the Start menu and the desktop, and the `askone` command to the PATH: open a **new** terminal to
use it.

### "Windows protected your PC"

The installer is not signed yet, so Windows warns before running it:

1. Your browser may say the file is not commonly downloaded. Choose **Keep** (in Edge: **...**, then **Keep**, then
   **Keep anyway**).
2. When you open it, SmartScreen shows **Windows protected your PC**. Click **More info**, check that the file is
   `Askone setup v<version>.msi`, then **Run anyway**.

If **Smart App Control** is on (Windows 11, *Windows Security > App & browser control*), it blocks unsigned installers
outright and offers no "run anyway". Askone cannot be installed while it is on.

### Silent install

```text
msiexec /i "Askone setup v<version>.msi" /qn                      just for you, no administrator rights
msiexec /i "Askone setup v<version>.msi" /qn ALLUSERS=1           for everyone, from an elevated prompt
```

## First run

The window shows the terms of use once, until you click **Got it**.

### 1. Create your Spotify application (once)

Spotify lets a personal application serve only 5 people, and gives more only to established companies, so Askone
cannot come with one: each person creates their own. It takes a few minutes, and the Spotify account that creates
it needs an active **Premium** subscription.

1. Sign in to the [Spotify developer dashboard](https://developer.spotify.com/dashboard) with your Spotify account,
   and accept the developer terms if asked.
2. Choose **Create app**. Give it any name and description (for example *Askone*), and under **Redirect URIs** add
   exactly `http://127.0.0.1:5000/callback`.
3. Under **Which API/SDKs are you planning to use?**, tick **Web API**. Accept the terms and **Save**.
4. Open the application's **Settings** and copy its **Client ID**.
5. In Askone, click **Settings** and paste it in **Client id**. On the command line:
   `askone config --spotify-client-id <client id>`.

Someone else can use your application if you add their Spotify name and email under the application's **User
Management**, up to 5 people in all.

### 2. Sign in

Click **Sign in** in the window, or run `askone login`. Your browser opens on Spotify's consent page; once you agree,
it comes back to Askone and the window shows your Spotify name. The session is kept, encrypted for your Windows
account, until you sign out.

If the browser shows `INVALID_CLIENT: Invalid redirect URI`, the redirect URI of step 2 is missing or differs: it has
to be `http://127.0.0.1:5000/callback`, character for character.

### 3. List and download

In Spotify, open the playlist, then **...** > **Share** > **Copy link to playlist**. Paste the link in Askone and click
**List**: the tracks appear, all ticked. Untick the ones you do not want and click **Download**. The tracks go into
`Music\Askone` unless you choose another **Download folder** in the settings. Each playlist gets a folder of its own,
and a track already there is not downloaded again.

When a run is over, **Retry failed** tries the failed tracks again. Askone remembers each track's last result, so this
works after a restart too.

### Playlists you do not own

Spotify only lets a personal application read the tracks of **your own playlists and of the ones you collaborate on**.
A friend's playlist, a public one or one made by Spotify (Discover Weekly, editorial playlists) is refused with a
message saying so. To download it, copy it into a playlist of yours: open it in Spotify, select all its tracks
(Ctrl+A), right-click and choose **Add to playlist** > **New playlist**, then give Askone the link of the copy.

## When YouTube asks whether you are a robot

YouTube watches how many videos one internet address fetches. Past a certain number it answers "Sign in to confirm
you're not a bot", and Askone reports the track as refused. This is about your connection, not about Askone, and it
wears off after a while. Askone already does what it can: one track at a time by default, a pause between requests,
IPv4 only, and a pause of its own when YouTube starts refusing.

If it keeps happening, let yt-dlp use your YouTube cookies:

- **Firefox** (recommended): sign in to YouTube in Firefox, then choose Firefox under **Cookies from browser** in
  the settings, or run `askone config --youtube-cookies-from-browser firefox`.
- **Other browsers**: Chrome, Edge and other Chromium browsers encrypt their cookies on Windows in a way yt-dlp cannot
  read. Export your YouTube cookies to a file with an extension such as *Get cookies.txt LOCALLY*, then choose that file
  under **Cookies file** in the settings, or run `askone config --youtube-cookies-file <path to cookies.txt>`.

Cookies are never used unless you turn them on. Downloading several tracks at once (**Tracks at a time** in the
settings) makes the check come sooner.

## Command line

```text
askone list     <playlist link>                 show the tracks
askone download <playlist link>                 download into Music\Askone\<playlist>
askone download <link> -o D:\Music -f flac      choose the folder and the format (mp3, m4a, opus, flac)
askone download <link> -t 3,7-9                 only some positions
askone download <link> --retry-failed           only the tracks whose last run failed
askone history                                  what Askone remembers of past runs
askone config                                   show the settings; add options to change them
askone login | logout | about
```

`askone <command> --help` lists every option. Results go to the standard output and messages to the error output, so
the output can be piped. Exit codes, for scripts:

| Code | Meaning |
| --- | --- |
| 0 | Done; every requested track has a file |
| 1 | Unknown command or option, or a value outside the listed choices |
| 2 | Not a Spotify playlist link |
| 3 | The run finished, but some tracks have no file |
| 5 | Playlist missing, unreadable, or not yours |
| 6 | Spotify refused the sign-in, the session or the application |
| 7 | No Spotify client id configured |
| 8 | A value that is refused: out of range, missing file, conflicting options |
| 9 | Not signed in; run `askone login` |
| 10 | Unexpected error |
| 11 | A service could not be reached; try again later |
| 12 | YouTube kept refusing this connection; wait, or configure cookies |

## Updating

Askone looks at this page at most once a day and says in its status bar when a newer version is out. Download the new
installer and run it: it replaces the installed version and keeps your settings, session and history. Choose the same
option (just for you, or for everyone) as the first time.

You can choose which releases it tells you about under **Tell me about new versions** in the settings, or with
`askone config --update-channel`: `stable`
(the default), `prerelease` to hear about test versions too, or `off`.

## Uninstalling

Uninstall **Askone** from *Settings > Apps > Installed apps*. The program, its shortcuts and its PATH entry go away;
your downloaded music, settings, history and the downloaded tools stay. To remove them too, delete the folders listed
below. To withdraw Askone's access to your Spotify account, remove it under
[Apps](https://www.spotify.com/account/apps/) in your Spotify account, and delete the application on the developer
dashboard if you no longer need it.

## Where Askone keeps its files

| What | Where |
| --- | --- |
| Downloaded music | `Music\Askone` by default, one folder per playlist |
| Settings and Spotify session (encrypted) | `%APPDATA%\Prometeo\Askone\settings.json` |
| History of each playlist's results | `%LOCALAPPDATA%\Prometeo\Askone\history` |
| Logs, kept 7 days | `%LOCALAPPDATA%\Prometeo\Askone\logs` |
| yt-dlp, Deno, ffmpeg | `%LOCALAPPDATA%\Prometeo\Askone\tools` |
| Streams being converted | `%LOCALAPPDATA%\Prometeo\Askone\work` |

The window and the command line share all of them, so a setting changed in one applies to the other.

## When something goes wrong

Askone's messages say what to do for the common cases: no client id, a playlist you do not own, a Spotify account
without Premium or not on the application's user list, YouTube's robot check. For anything else, the log of the day
is in `%LOCALAPPDATA%\Prometeo\Askone\logs`; it holds no password or token, and is what to look at, or send, first.

The interface speaks English, French and Spanish, following Windows; change it in the settings or with
`askone config --language fr`.

---

Licence: Apache 2.0. Published by Prometeo. The third-party components and their licences are listed in the About
window and by `askone about`.

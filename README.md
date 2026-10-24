# Askone releases

Askone downloads the tracks of a Spotify playlist, from a desktop window or the `askone` command line, on Windows.

This repository only publishes the installers. Download `Askone setup v<version>.msi` from the
[Releases](https://github.com/PrometeoConseil/Askone-releases/releases) page. The .NET runtime is inside it, so nothing
else has to be installed. The installer asks whether to install for you alone (no administrator rights) or for every
user of the machine.

Askone checks this page at most once a day and tells you in its status bar when a newer version is published. That
check can be turned off in the settings, or with `askone config --update-channel off`.

Askone needs your own Spotify developer application. The settings window lists the steps to create one, and
`askone list` prints them when the client id is missing.

Licence: Apache 2.0. Published by Prometeo.

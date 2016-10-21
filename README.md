# gdx-deploy-sample
Sample Gradle configurations for LibGDX for preparing and deploying release builds

# Overview
This project includes commands to obfuscate the source using Proguard, generate builds via Packr, set an icon on Windows executables (currently not supported by Packr), and push the builds to itch.io, all in a single command. It is currently setup to generate builds for Windows, Mac, and Linux, with 32-bit and 64-bit variants for Windows and Linux.

# Setup

Download Proguard: http://proguard.sourceforge.net/

Download Packr: https://github.com/libgdx/packr

Download Resource Hacker: http://www.angusj.com/resourcehacker/

Install Butler: https://itch.io/docs/butler/installing.html

In your main build.gradle file, add this under buildscript -> dependencies:

```groovy
classpath 'de.undercouch:gradle-download-task:3.1.1'
classpath 'org.mini2Dx:butler:1.0.1'
```

Then under `project(:desktop)`, add:

```groovy
apply plugin: "org.mini2Dx.butler"

butler {
  user = "your-itch-io-username"
  game = "your-itch-io-game"
}
```

Then grab the build.gradle file in the desktop folder from here and put it in desktop/build.gradle. After that, you'll need to updae the paths for `dekstopWorkingDir` and `utilsDir`, then put proguard.jar, packr.jar and ResourceHacker.exe into the utils folder specified. You'll also want to replace the various references in this file to Questionable Markup, which was the name of my game.

Add a Proguard configuration file to your LibGDX project under desktop/proguard-project.txt. A sample is included in this repo, although you'll need to update the packages to correspond to class names for your game.

Add a configuration file for Packr in your LibGDX project under desktop/config/packr-minimize.json. There's also a sample of this in the repo files.

Finally, add the icons you want to use for your Windows / Mac builds to destkop/assets/icons/app-icon.ico and desktop/assets/icons/app-icon.icns, respectively.

# Obfuscate Code (Proguard)

Proguard is used to obfuscate your game's source code, which should make it more difficult for others to decompile. To run Proguard and obfuscate your desktop build, execute:

```
gradlew desktop:obfuscate
```

This will invoke `desktop:dist` to ensure you have a build to obfuscate, then run Proguard to obfuscate it. Currently, this assumes that the file generated by `desktop:dist` will be called desktop-1.0.jar. If you changed the version number of your game in your main build.gradle file, you'll probably need to tweak this.

# Generate Executables (Packr)

Packr is used to generate an executeable for the jar file. To generate all builds using Packr, execute:

```
gradlew desktop:pack
```

This will take the obfuscated build generated from the previous step as an input. Invoking `pack` will automatically run the `obfuscate` so you don't need to execute that separately. You can also run the process individually for each build:

```
gradlew desktop:packWindows32
gradlew desktop:packWindows64
gradlew desktop:packMac
gradlew desktop:packLinux32
gradlew desktop:packLinux64
```

Running the build-specific commands will NOT execute `obfuscate` first.

# Set Executable Icons (Resource Hacker)

Packr doesn't currently have the ability to set icons on Windows builds, so we use Resource Hacker to fill that gap. Execute the following to set the icon on both Windows 32 and 64 bit builds:

```
gradlew desktop:setIcon
```

This will invoke all steps up to this point as well. Alternatively, you can also execute the command on each build separately:

```
gradelw desktop:setIconWindows32
gradlew desktop:setIconWindows64
```

Running the build-specific commands will NOT execute `pack` first.

# Upload to itch.io (Butler)

Butler is a command-line tool that can be used to upload builds to itch.io. The upload all builds, execute:

```
gradlew desktop:push
```

This will invoke all prior steps as well, so you can use this command to run the entire process. Alternatively, you can also run the command individually for each build:

```
gradlew desktop:pushWindows32
gradlew desktop:pushWindows64
gradlew desktop:pushMac
gradlew desktop:pushLinux32
gradlew desktop:pushLinux64
```

As with the other build-specific commands, these commands will NOT execute any previous steps.

# Other Output Options

If you're not using itch.io, you can use the `release` command to run all steps up until that point:

```
gradlew desktop:release
```

This command will essentially do the same thing as `setIcon`, it just puts a prettier name on it.

# Caveats
* These commands were put together with an extremely limited knowledge of Gradle. I know thre's better ways to push together these commands and integrate them into a LibGDX project. This project mainly exists to deomnstrate the set of commands I was using the generate and deploy builds, but any suggestoins on how to improve things are greatly appreciated!

* There currently isn't anything setup to check whether or not anything has changed since the last build, so unlike some other LibGDX Gradle commands, all of the commands included here will execute each time their run, even if the inputs haven't changed. I did some research into how to fix this, but I wasn't able to figure it out. Any insight would be awesome.

* I couldn't figure out how to properly re-use some pieces from the Butler plugin, so I threw my hands up and did some copy-pasta to get it working the way I wanted it to. Hopefully someone can come up wit a better way to do that.

* Resource Hacker can only be run from a Windows machine or VM, since it uses Windows APIs to set the icon on the executable. There may be other solutions out there to do this from Mac and Linux, however.

* This was only built with desktop builds in mind, since that's what I was focused on for my project. Additional work would be needed to apply this to Android, iOS, or HTML builds.

### Embrace Gradle configuration caching issue

A minimal Android project demonstrating a Gradle configuration caching issue introduced by the Embrace plugin.

### Steps to reproduce

* `./gradlew :app:assembleDebug` 
    * A configuration cache entry will be stored.
* `./gradlew :app:assembleDebug`
    * The configuration cache is be reused.
* `touch ../foobarbaz`
    * Create an unrelated file in the parent directory of the project
* `./gradlew :app:assembleDebug` 
    * The configuration cache will *NOT* be reused.
    * Gradle logs "Calculating task graph as configuration cache cannot be reused because directory '..' has changed."

### What happens

If the Embrace plugin is applied, changes in the parent of the project directory cause Gradle 
configuration cache misses:

```
dxf2445 ~/dev/EmbraceCaching [main]
$ ./gradlew :app:assembleDebug
Calculating task graph as no cached configuration is available for tasks: :app:assembleDebug

> Task :app:stripDebugDebugSymbols
Unable to strip the following libraries, packaging them as they are: libembrace-native.so.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.7/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 3s
37 actionable tasks: 37 executed
Configuration cache entry stored.

dxf2445 ~/dev/EmbraceCaching [main]
$ ./gradlew :app:assembleDebug
Reusing configuration cache.

BUILD SUCCESSFUL in 1s
37 actionable tasks: 6 executed, 31 up-to-date
Configuration cache entry reused.
dxf2445 ~/dev/EmbraceCaching [main]
$ touch ../foobarbaz
dxf2445 ~/dev/EmbraceCaching [main]

$ ./gradlew :app:assembleDebug
Calculating task graph as configuration cache cannot be reused because directory '..' has changed.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.7/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 1s
37 actionable tasks: 6 executed, 31 up-to-date
Configuration cache entry stored.
```

### What should happen

If the Embrace plugin is not applied (comment out line 6 in app/build.gradle.kts), caching works as expected:

```
dxf2445 ~/dev/EmbraceCaching [main]
$ ./gradlew :app:assembleDebug
Calculating task graph as configuration cache cannot be reused because file 'app/build.gradle.kts' has changed.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.7/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 2s
35 actionable tasks: 19 executed, 16 up-to-date
Configuration cache entry stored.

dxf2445 ~/dev/EmbraceCaching [main]
$ ./gradlew :app:assembleDebug
Reusing configuration cache.

BUILD SUCCESSFUL in 374ms
33 actionable tasks: 33 up-to-date
Configuration cache entry reused.

dxf2445 ~/dev/EmbraceCaching [main]
$ touch ../bazbarfoo
dxf2445 ~/dev/EmbraceCaching [main]
$ ./gradlew :app:assembleDebug
Reusing configuration cache.

BUILD SUCCESSFUL in 426ms
33 actionable tasks: 33 up-to-date
Configuration cache entry reused.
dxf2445 ~/dev/EmbraceCaching [main]
```
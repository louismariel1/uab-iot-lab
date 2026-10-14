I recommend starting with `edge/android/settings.gradle.kts`, the first file in the implementation order. It establishes the Android Gradle project and makes the `app` module available to the build system.

## File 1 — `edge/android/settings.gradle.kts`

Use this initial configuration:

Writing

```
// edge/android/settings.gradle.kts

pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)

    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "SSPEdge"
include(":app")
```

### Validation

- Declares the `app` module at `edge/android/app`.
- Configures repositories for Android and Kotlin Gradle plugins.
- Centralizes dependency repositories at the project level.
- Does not introduce unnecessary dependencies.

Status: Proposed configuration; not yet built or verified.

## Next implementation sequence

After adding this file, proceed in this order:

1. `edge/android/build.gradle.kts` — select compatible Android Gradle Plugin and Kotlin plugin versions.
2. `edge/android/app/build.gradle.kts` — configure the Android SDK, Java/Kotlin compatibility, and test dependencies.
3. `edge/android/app/src/main/AndroidManifest.xml` — declare application and BLE permissions.
4. Protocol and state-machine unit tests.
5. Android BLE client implementation.

One important detail: the Gradle wrapper is currently a placeholder. We should generate a real wrapper after selecting compatible Gradle and Android Gradle Plugin versions, rather than trying to run the placeholder.

For now, add the first file and then we can validate the root `build.gradle.kts` configuration.

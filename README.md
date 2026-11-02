# Android Build System – From Source Code to Google Play Store

## 1. Introduction

When we write an Android application using Kotlin or Java, the source code cannot run directly on an Android device.

It must go through a build process that converts the source code into a format Android understands, packages the required resources, and signs the final application.

During this process, several technologies and tools work together:

- **JDK:** Provides Java development tools, including the Java compiler.
- **JVM:** Executes Java bytecode during development and in JVM-based environments.
- **Bytecode:** Intermediate instructions generated from Java or Kotlin source code.
- **D8 / R8:** Convert JVM bytecode into Android DEX bytecode.
- **ART:** Android Runtime, which executes Android applications on modern Android devices.
- **JIT / AOT:** Runtime compilation techniques used by ART.
- **Android SDK:** Provides the APIs and tools required to develop Android applications.
- **Gradle:** Build automation system that coordinates the build.
- **Android Gradle Plugin (AGP):** Adds Android-specific build tasks to Gradle.

Let's understand where each component fits.

---

## 2. Big Picture: Android Development and Execution Flow

There are two important processes:

1. **Build time:** Converts source code into an installable APK or Android App Bundle.
2. **Runtime:** Executes the installed application on an Android device.

### Build-time flow

```text
Developer writes Kotlin / Java code
              |
              v
       JDK / Kotlin Compiler
              |
              v
        JVM Bytecode (.class)
              |
              v
          D8 / R8
              |
              v
       Android DEX (.dex)
              |
              v
   APK Packaging + Resources
              |
              v
          ZIPALIGN
              |
              v
          APK Signing
              |
              v
        APK / AAB Output
              |
              v
       Google Play Store
```

### Runtime flow

```text
User installs the application
              |
              v
      Android Operating System
              |
              v
       Android Runtime (ART)
              |
              v
       Executes DEX bytecode
              |
              v
        JIT / AOT Compilation
              |
              v
       Application runs on device
```

**Important:** JVM bytecode and Android DEX bytecode are different formats. Android applications normally execute DEX bytecode through ART, not through a standard JVM.

---

## 3. JDK, JVM and Bytecode

### What is JDK?

JDK stands for Java Development Kit.

It provides the tools required to develop Java applications, including:

- Java compiler (`javac`)
- Java runtime components
- Development utilities
- Debugging and packaging tools

**Where is it used?**

During development and the build process, Java compilation tools from the JDK can compile Java source code into JVM bytecode.

Android Studio and Gradle also use a configured JDK to run the Gradle build process.

### What is JVM?

JVM stands for Java Virtual Machine.

It is a virtual machine designed to execute Java bytecode.

For example:

```text
Java Source Code
       |
       v
     javac
       |
       v
JVM Bytecode (.class)
       |
       v
      JVM
       |
       v
Program Execution
```

**Where is it used in Android development?**

- Gradle and many Android build tools run on the JVM.
- Java and Kotlin source code are compiled into JVM-compatible bytecode before Android-specific conversion.
- The Android application itself does not normally run on a standard JVM.

### What is Bytecode?

Bytecode is an intermediate instruction format generated after compiling source code.

For example:

```text
MainActivity.java
       |
       v
Java Compiler (javac)
       |
       v
MainActivity.class
```

Similarly, Kotlin code is compiled into JVM-compatible `.class` files.

```text
MainActivity.kt
       |
       v
Kotlin Compiler
       |
       v
MainActivity.class
```

The `.class` file contains JVM bytecode.

Android's D8 or R8 tool then converts the compiled bytecode into DEX format.

---

## 4. DVM, ART, JIT and AOT

These concepts are related to executing Android applications on devices.

### What is DVM?

DVM stands for Dalvik Virtual Machine.

It was the runtime used by older Android versions to execute Android applications.

- It executed DEX bytecode.
- It was designed specifically for Android devices.
- It used a register-based architecture.
- It used just-in-time compilation in later versions.

Dalvik was replaced by ART starting with Android 5.0 (Lollipop).

### What is ART?

ART stands for Android Runtime.

It is the runtime environment used by modern Android versions to execute applications.

Its responsibilities include:

- Executing DEX bytecode.
- Memory management and garbage collection.
- Application runtime support.
- Compilation and optimization of application code.

**Where does ART come in?**

ART comes into the picture after the application has been installed on an Android device and Android starts executing it.

### What is JIT?

JIT stands for Just-In-Time compilation.

It compiles frequently executed bytecode into machine code while the application is running.

For example:

```text
DEX Bytecode
     |
     v
ART observes frequently used code
     |
     v
JIT Compiler
     |
     v
Native Machine Code
     |
     v
Faster execution
```

### What is AOT?

AOT stands for Ahead-Of-Time compilation.

It compiles application code into native machine code ahead of execution.

ART can use profile information to identify frequently used code and compile it ahead of time.

### JIT vs AOT

| JIT | AOT |
|---|---|
| Compiles code while the app runs | Compiles code ahead of execution |
| Uses runtime information | Can use collected profile information |
| Compilation happens during execution | Compilation happens before the code is executed |
| Can introduce compilation overhead during runtime | Can reduce runtime compilation overhead |

Modern ART uses a combination of JIT and profile-guided AOT compilation.

**Important:** D8/R8 produces DEX bytecode during the Android build. ART handles execution and runtime compilation on the device.

---

## 5. Android SDK and Its Components

SDK stands for Software Development Kit.

The Android SDK provides the tools, libraries, APIs and platform components required to develop, build, debug and test Android applications.

It contains several important components.

### 5.1 SDK Tools / Command-Line Tools

These provide utilities to manage Android SDK packages and development environments.

Examples:

- `sdkmanager`: Installs and manages SDK packages.
- `avdmanager`: Creates and manages Android Virtual Devices.
- Emulator: Runs virtual Android devices.

**When are they used?**

During SDK setup, package management, emulator creation and development environment configuration.

### 5.2 Platform Tools

Platform Tools provide tools that communicate with Android devices and emulators.

Important examples:

- **ADB (Android Debug Bridge):** Installs applications, runs shell commands, reads logs and communicates with devices.
- **Fastboot:** Used for supported device bootloader and flashing operations.

Example:

```bash
adb devices
adb install app-debug.apk
adb logcat
```

**When are they used?**

During development, debugging, testing and installing applications on physical devices or emulators.

### 5.3 Android Platform

An Android Platform package contains the Android framework APIs for a particular Android API level.

For example:

- Android 14 – API 34
- Android 15 – API 35

It provides the Android APIs against which the application is compiled.

**Where does it come in?**

When configuring `compileSdk` in the app's Gradle file.

```kotlin
android {
    compileSdk = 35
}
```

This means the application can compile against the Android 15 APIs.

### 5.4 Build Tools

Build Tools provide utilities required to convert application source code and resources into an APK or AAB.

Important tools include:

| Tool | Responsibility |
|---|---|
| AAPT2 | Compiles and links Android resources |
| D8 | Converts JVM bytecode into DEX |
| R8 | Shrinks, optimizes, obfuscates and converts code into DEX |
| zipalign | Aligns data inside APK files |
| apksigner | Signs APK files |

**When are they used?**

During the Android build process. Gradle and AGP invoke the appropriate tools through their build tasks.

---

## 6. Gradle and Android Gradle Plugin (AGP)

### What is Gradle?

Gradle is a build automation system.

It manages the Android build process, including:

- Compiling source code.
- Processing resources.
- Managing dependencies.
- Running tests.
- Packaging APKs and AABs.
- Running build tasks.

Gradle uses a configuration file such as:

```text
build.gradle
build.gradle.kts
```

### What is Android Gradle Plugin?

AGP stands for Android Gradle Plugin.

It is a plugin that connects Android-specific build functionality with Gradle.

It provides Android build tasks such as:

```bash
./gradlew assembleDebug
./gradlew assembleRelease
./gradlew bundleRelease
./gradlew installDebug
```

**Difference:**

- Gradle is the general build automation system.
- AGP provides Android-specific build functionality.
- Android SDK Build Tools provide the actual utilities used by many of those tasks.

### Gradle Build Lifecycle

Gradle executes the build in three main phases:

1. **Initialization:** Identifies the project and its modules.
2. **Configuration:** Reads build scripts, configures plugins and creates tasks.
3. **Execution:** Runs the requested tasks and their dependencies.

### Groovy DSL vs Kotlin DSL

Gradle supports two common DSLs for writing build scripts.

**Groovy DSL:**

```groovy
plugins {
    id 'com.android.application'
}
```

File:

```text
build.gradle
```

**Kotlin DSL:**

```kotlin
plugins {
    id("com.android.application")
}
```

File:

```text
build.gradle.kts
```

Both configure Gradle. Kotlin DSL provides stronger IDE support and Kotlin-based syntax.

---

## 7. Android Project Configuration

An Android application commonly has configuration in its module-level `build.gradle` or `build.gradle.kts` file.

### compileSdk

Specifies the Android API level used to compile the application.

```kotlin
compileSdk = 35
```

### minSdk

Specifies the minimum Android API level supported by the application.

```kotlin
defaultConfig {
    minSdk = 23
}
```

### targetSdk

Specifies the Android API level whose platform behavior the application targets.

```kotlin
targetSdk = 35
```

### versionCode and versionName

```kotlin
defaultConfig {
    versionCode = 2
    versionName = "1.1"
}
```

- `versionCode`: Integer used by Google Play to identify and order application versions. Increase it for each new Play Store release.
- `versionName`: Human-readable version shown to users.

### Build Types

Build types define different configurations for building the application.

Common examples:

- Debug
- Release

```kotlin
buildTypes {
    debug {
        applicationIdSuffix = ".debug"
    }

    release {
        isMinifyEnabled = true
    }
}
```

### Product Flavors

Product flavors allow us to create different editions of the same application.

For example:

- Free
- Paid

### Build Variants

A build variant is created by combining a product flavor with a build type.

For example:

```text
freeDebug
freeRelease
paidDebug
paidRelease
```

---

## 8. Dependencies, Maven and Repositories

Android applications often depend on external libraries.

Examples:

- Retrofit
- OkHttp
- Room
- Hilt
- Firebase

Gradle downloads these dependencies from configured repositories.

Example:

```kotlin
repositories {
    google()
    mavenCentral()
}
```

### What is Maven?

Maven is a build automation and dependency management system commonly used in Java projects.

Gradle and Maven are separate build systems.

| Gradle | Maven |
|---|---|
| Uses Groovy or Kotlin DSL | Uses XML configuration |
| Flexible task-based build system | Convention-based lifecycle |
| Supports incremental and parallel execution | Provides a structured build lifecycle |
| Commonly used in Android projects | Commonly used in Java projects |

**Important distinction:**

Maven Central is a dependency repository. It is not the same thing as the Maven build system.

---

## 9. Multi-Module Architecture

Android projects can be divided into multiple modules.

For example:

```text
MyApplication
│
├── app
│
├── core
│   ├── common
│   ├── network
│   └── database
│
├── feature
│   ├── login
│   └── dashboard
│
└── domain
```

### Types of modules

- Application module: Contains the main application and produces the APK/AAB.
- Android library module: Contains reusable Android-specific code and resources.
- Kotlin/Java library module: Contains reusable code without requiring Android framework APIs.

### Benefits

- Better separation of responsibilities.
- Easier testing and code maintenance.
- Code reuse across features.
- Improved team collaboration.
- Incremental builds can avoid rebuilding unaffected tasks.
- Independent tasks may execute in parallel when their dependencies allow it.

### Disadvantages

- More Gradle and dependency configuration.
- Inter-module communication needs careful design.
- Cross-module refactoring can become more complex.
- Additional overhead for small applications.

---

## 10. Actual Android APK Build Process

Now let's understand what happens when we execute:

```bash
./gradlew assembleDebug
```

Gradle asks AGP to execute the required Android build tasks.

### Step 1: Resource Processing – AAPT2

AAPT2 stands for Android Asset Packaging Tool 2.

It handles Android resources such as:

```text
res/
├── drawable/
├── layout/
├── mipmap/
├── values/
└── xml/
```

It has two main operations.

**A. Compile**

AAPT2 compiles individual Android resources into intermediate `.flat` files.

```text
res/layout/activity_main.xml
              |
              v
            AAPT2
              |
              v
activity_main.xml.flat
```

**B. Link**

AAPT2 links the compiled resources with the manifest and other resource information.

It produces resource-related outputs, including the resource table and compiled resource package. AGP also generates the resource symbols/classes used by application code.

For example:

```kotlin
setContentView(R.layout.activity_main)
```

The `R` class provides references to application resources.

**Note:** The `assets/` directory is generally packaged as raw assets rather than compiled like resources under `res/`.

### Step 2: Java and Kotlin Compilation

Java source files are compiled using Java compiler tools.

Kotlin source files are compiled using the Kotlin compiler.

```text
MainActivity.java
       |
       v
Java Compiler
       |
       v
MainActivity.class
```

```text
MainActivity.kt
       |
       v
Kotlin Compiler
       |
       v
MainActivity.class
```

The generated `.class` files contain JVM-compatible bytecode.

### Step 3: D8 or R8 – DEX Conversion

Android devices execute DEX bytecode rather than standard JVM `.class` bytecode.

D8 converts JVM bytecode into DEX format.

```text
.class files
     |
     v
     D8
     |
     v
classes.dex
```

**What is R8?**

R8 is used for code shrinking, optimization and obfuscation when configured.

It can also perform DEX conversion, replacing the normal D8 processing in that build configuration.

Example:

```kotlin
buildTypes {
    release {
        isMinifyEnabled = true
        isShrinkResources = true
    }
}
```

R8 can:

- Remove unused code.
- Optimize code.
- Obfuscate class and method names.
- Generate DEX output.

**D8 vs R8:**

| D8 | R8 |
|---|---|
| Converts JVM bytecode into DEX | Shrinks, optimizes and can obfuscate code |
| Normal DEX conversion tool | Can replace D8 in configured builds |
| Does not perform general code shrinking | Performs code shrinking and optimization |

### Step 4: ProGuard Rules

ProGuard is an older tool used for code shrinking and obfuscation.

R8 supports ProGuard-compatible configuration rules.

For example:

```proguard
-keep class com.example.MyClass {
    *;
}
```

These rules tell R8 which classes or members should be preserved.

### Step 5: Multidex

A single DEX file has a limit of 65,536 method references.

If the application exceeds this limit, multiple DEX files may be required.

Example:

```text
classes.dex
classes2.dex
classes3.dex
```

**What happens on different Android versions?**

- Android 5.0 (API 21) and above: ART natively supports multidex.
- Android 4.4 (API 20) and below: Multidex support library/configuration is required when the limit is exceeded.

### Step 6: APK Packaging

AGP packages the required application components into an APK.

An APK generally contains:

```text
app-debug.apk
│
├── AndroidManifest.xml
├── classes.dex
├── resources.arsc
├── res/
├── assets/
├── lib/
└── META-INF/
```

The exact contents vary by application and build configuration.

### Step 7: ZIPALIGN

`zipalign` aligns uncompressed data within the APK to improve access efficiency.

It does not simply compress the APK.

### Step 8: APK Signing – apksigner

Android requires APKs to be digitally signed before installation.

`apksigner` signs the APK using a signing key.

There are two common signing configurations:

- Debug signing: Uses the automatically generated debug keystore.
- Release signing: Uses the configured release keystore.

**Important:** APK alignment is performed before signing in the standard signing workflow.

### Step 9: Final APK

After packaging, alignment and signing, the APK is ready for installation or distribution.

For example:

```text
app/build/outputs/apk/debug/app-debug.apk
```

---

## 11. Debug APK vs Release APK

| Debug APK | Release APK |
|---|---|
| Used during development and testing | Used for distribution |
| Signed with debug key by default | Signed with configured release key |
| Usually includes debugging support | Usually optimized for production |
| Normally not minified by default | Can enable R8 shrinking and obfuscation |

---

## 12. APK vs AAB

### What is APK?

APK stands for Android Package Kit.

It is the installable Android application package.

### What is AAB?

AAB stands for Android App Bundle.

It is a publishing format uploaded to Google Play.

Instead of uploading one universal APK, developers upload an AAB. Google Play uses it to generate optimized APKs for users' devices.

### AAB delivery flow

```text
Developer builds AAB
        |
        v
   app-release.aab
        |
        v
  Upload to Google Play
        |
        v
Google Play processes bundle
        |
        v
Generates optimized APK splits
        |
        v
Delivers required APKs
        |
        v
User installs application
        |
        v
ART executes DEX code
```

### What are APK splits?

Google Play can generate different APK components based on device requirements.

Examples:

- Base APK: Main application code and required resources.
- Configuration APKs: Device-specific resources such as language, screen density or CPU architecture.
- Dynamic feature APKs: Optional feature modules delivered when required.

The Android package installer installs the required splits together as one application.

### What is Play App Signing?

Google Play App Signing manages the app signing key used for APKs delivered to users.

The developer signs the uploaded app bundle using an upload key. Google Play verifies the upload and signs the generated APKs using the app signing key.

---

## 13. Important Android Build Commands

```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Build release AAB
./gradlew bundleRelease

# Install debug APK on connected device
./gradlew installDebug

# Run unit tests
./gradlew test

# Clean build outputs
./gradlew clean

# List available Gradle tasks
./gradlew tasks
```

---

## 14. Complete Android Build and Runtime Lifecycle

This diagram combines the tools and technologies discussed above.

```text
                 DEVELOPMENT
                      |
                      v
           Android Studio / IDE
                      |
                      v
             Kotlin / Java Code
                      |
                      v
             Gradle + AGP
                      |
          +-----------+-----------+
          |                       |
          v                       v
     Resource Files          Source Files
          |                       |
          v                       v
        AAPT2              JDK / Kotlin Compiler
          |                       |
          v                       v
   Compiled Resources        JVM Bytecode
          |                    (.class)
          |                       |
          |                       v
          |                    D8 / R8
          |                       |
          |                       v
          |                  DEX Bytecode
          |                       |
          +-----------+-----------+
                      |
                      v
                APK Packaging
                      |
                      v
                  ZIPALIGN
                      |
                      v
                 APK Signing
                      |
                      v
                 APK / AAB
                      |
                      v
                Google Play
                      |
                      v
             APK Installation
                      |
                      v
             Android Operating System
                      |
                      v
                  ART Runtime
                      |
              +-------+-------+
              |               |
              v               v
             JIT             AOT
              |               |
              +-------+-------+
                      |
                      v
              Native Execution
                      |
                      v
              Application Runs
```

**Where do the Android SDK components fit?**

| SDK component | When it is involved |
|---|---|
| SDK Command-Line Tools | SDK installation and package management |
| Platform Tools (ADB) | Device communication, debugging and installation |
| Android Platform | Compile-time Android APIs selected through `compileSdk` |
| Build Tools | Resource processing, DEX conversion, alignment and signing |
| Emulator | Running and testing applications on virtual Android devices |

---

## 15. Quick Revision Table

| Term | Meaning | Main purpose |
|---|---|---|
| JDK | Java Development Kit | Provides Java development tools |
| JVM | Java Virtual Machine | Executes JVM bytecode |
| Bytecode | Intermediate instructions | Output of Java/Kotlin compilation |
| DEX | Dalvik Executable | Android bytecode format |
| DVM | Dalvik Virtual Machine | Older Android runtime |
| ART | Android Runtime | Executes Android applications |
| JIT | Just-In-Time | Compiles code during execution |
| AOT | Ahead-Of-Time | Compiles code ahead of execution |
| SDK | Software Development Kit | Android development tools and APIs |
| SDK Tools | Command-line utilities | Manage SDK packages and emulators |
| Platform Tools | ADB and related tools | Communicate with devices |
| Android Platform | API-level framework | Provides Android APIs |
| Build Tools | AAPT2, D8, R8, etc. | Build and package Android apps |
| Gradle | Build automation system | Coordinates build tasks |
| AGP | Android Gradle Plugin | Adds Android-specific Gradle tasks |
| AAPT2 | Android Asset Packaging Tool 2 | Processes Android resources |
| D8 | DEX compiler | Converts JVM bytecode into DEX |
| R8 | Code optimizer | Shrinks, optimizes and obfuscates code |
| ProGuard | Code shrinking/obfuscation tool | Older alternative; rules supported by R8 |
| ZIPALIGN | APK alignment tool | Aligns APK data |
| apksigner | APK signing tool | Digitally signs APKs |
| APK | Android Package Kit | Installable application package |
| AAB | Android App Bundle | Google Play publishing format |

---

## Conclusion

The Android build process starts with Kotlin or Java source code. JDK and Kotlin compiler tools compile the source code into JVM-compatible bytecode. D8 or R8 converts that bytecode into Android DEX format, while AAPT2 processes application resources.

Gradle and Android Gradle Plugin coordinate these operations using Android SDK components and Build Tools. The generated application is packaged, aligned and signed as an APK, or packaged as an AAB for Google Play.

After installation, **ART (Android Runtime)** executes the application's DEX bytecode. It uses JIT and profile-guided AOT compilation to optimize application execution.

Remember these three important distinctions:

1. **JVM vs ART:** JVM executes JVM bytecode; ART executes Android DEX bytecode.
2. **D8/R8 vs ART:** D8/R8 handle build-time bytecode conversion and optimization; ART handles runtime execution.
3. **Android SDK vs Build Tools vs Platform Tools:** SDK is the overall development kit; Build Tools help build applications; Platform Tools help communicate with and debug devices.

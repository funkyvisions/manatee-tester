Pull down my project on GitHub

It uses the 3rd party plugin (manateeworks-barcodescanner-v3) which IS compatible with cordova-android 6.4.0.

Execute the following:

    cordova prepare
    cordova build android

You'll see everything prepares and builds fine and you can see the libraries (.so files and jar) end up in the correct directories in the APK (view through Android Studio - Build -> Analyze APK).  See image1.

Now we are going to switch to cordova-android 7.1.0.

Execute the following:

    cp package.7.json package.json
    cp config.7.xml config.xml
    rm -rf platforms plugins
    cordova prepare
    cordova build android

Notice it fails. This is because the plugin.xml is incorrect (although the error would never lead you to discover this).  So let's fix it.  Unfortunately, I tried to fork the project and apply a patch, but there seems to be a bug with cordova-fetch (https://issues.apache.org/jira/browse/CB-13973), so instead I included it locally in this project along with the patch to plugin.xml.

I changed these lines in plugin.xml:

    <source-file src="src/android/libs/mwbscanner.jar" target-dir="libs" framework="true" />

    <source-file src="src/android/src/com/manateeworks/BarcodeScannerPlugin.java" target-dir="src/com/manateeworks" />
    <source-file src="src/android/src/com/manateeworks/ScannerActivity.java" target-dir="src/com/manateeworks" />
 
    <source-file src="src/android/res/layout/scanner.xml" target-dir="res/layout" />
    <source-file src="src/android/res/drawable/overlay_mw.png" target-dir="res/drawable" />
    <source-file src="src/android/res/drawable-hdpi/overlay_mw.png" target-dir="res/drawable-hdpi" />
    <source-file src="src/android/res/drawable/flashbuttonoff.png" target-dir="res/drawable" />
    <source-file src="src/android/res/drawable/flashbuttonon.png" target-dir="res/drawable" />
    <source-file src="src/android/res/drawable/zoom.png" target-dir="res/drawable" />
 
    <source-file src="src/android/libs/armeabi/libBarcodeScannerLib.so" target-dir="libs/armeabi" />
    <source-file src="src/android/libs/x86/libBarcodeScannerLib.so" target-dir="libs/x86" />
    <source-file src="src/android/libs/armeabi-v7a/libBarcodeScannerLib.so" target-dir="libs/armeabi-v7a" />
    <source-file src="src/android/libs/arm64-v8a/libBarcodeScannerLib.so" target-dir="libs/arm64-v8a" />
    <source-file src="src/android/libs/mips/libBarcodeScannerLib.so" target-dir="libs/mips" />

to this:

    <!-- For cordova-android 7 it seems I to have to put the jar file in libs for compile time and jniLibs for runtime -->
    <!-- Changed to lib-file -->
    <lib-file src="src/android/libs/mwbscanner.jar" />
    <resource-file src="src/android/libs/mwbscanner.jar" target="jniLibs/mwbscanner.jar" />

    <!-- no change -->
    <source-file src="src/android/src/com/manateeworks/BarcodeScannerPlugin.java" target-dir="src/com/manateeworks" />
    <source-file src="src/android/src/com/manateeworks/ScannerActivity.java" target-dir="src/com/manateeworks" />

    <!-- Changed to resource-file and used target instead of target-dir -->
    <resource-file src="src/android/res/layout/scanner.xml" target="res/layout/scanner.xml" />
    <resource-file src="src/android/res/drawable/overlay_mw.png" target="res/drawable/overlay_mw.png" />
    <resource-file src="src/android/res/drawable-hdpi/overlay_mw.png" target="res/drawable-hdpi/overlay_mw.png" />
    <resource-file src="src/android/res/drawable/flashbuttonoff.png" target="res/drawable/flashbuttonoff.png" />
    <resource-file src="src/android/res/drawable/flashbuttonon.png" target="res/drawable/flashbuttonon.png" />
    <resource-file src="src/android/res/drawable/zoom.png" target="res/drawable/zoom.png" />

    <!-- Changed to resource-file  and used target instead of target-dir (and changed to jniLibs) -->
    <resource-file src="src/android/libs/armeabi/libBarcodeScannerLib.so" target="jniLibs/armeabi/libBarcodeScannerLib.so" />
    <resource-file src="src/android/libs/x86/libBarcodeScannerLib.so" target="jniLibs/x86/libBarcodeScannerLib.so" />
    <resource-file src="src/android/libs/armeabi-v7a/libBarcodeScannerLib.so" target="jniLibs/armeabi-v7a/libBarcodeScannerLib.so" />
    <resource-file src="src/android/libs/arm64-v8a/libBarcodeScannerLib.so" target="jniLibs/arm64-v8a/libBarcodeScannerLib.so" />
    <resource-file src="src/android/libs/mips/libBarcodeScannerLib.so" target="jniLibs/mips/libBarcodeScannerLib.so" />

Now let's switch to the forked version of the plugin and see if it works.

Execute the following:

    cp package.fork.json package.json
    cp config.fork.xml config.xml
    rm -rf platforms plugins
    cordova prepare
    cordova build android

Now you see it builds once again and the APK is correct.  See image2.

However, the directives I put in plugin.xml are not backwards compatible with 6.4.0.  We can prove this by now switch back to cordova-android 6.4.0 and trying to build.

    cp package.6.json package.json
    cp config.6.xml config.xml
    rm -rf platforms plugins
    cordova prepare
    cordova build android

Now look at the APK and you will see the libraries are missing.  See image3.

In order to fix this, I have to create a mix on the old config plus the new.

See https://github.com/funkyvisions/phonegap-manateeworks-v3/commit/9f3b165832141e45fa2c2e92dbf62b2300f0caf2#diff-53f390d375398624afe1cfe1125f42bf

I don't think this is a very good solution.

There needs to be an approach that works with both.  I wonder if this (https://issues.apache.org/jira/browse/CB-8781) is missing from cordova-android 7.1.0:

    jniLibs.srcDirs = ['libs']

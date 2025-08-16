# حل مشاكل بناء مشروع Flutter

## المشاكل المحددة:
1. إصدار Gradle قديم (8.5.0) - يحتاج إلى 8.7.0 على الأقل
2. إصدار Android Gradle Plugin قديم (8.3.2) - يحتاج إلى 8.6.0 على الأقل  
3. عدم توافق إصدار Kotlin (2.1.0 vs 1.9.0)
4. مشكلة في تكوين CMake

## خطوات الحل:

### 1. تحديث إصدار Gradle

**الملف:** `F:\idoomfiber_app\android\gradle\wrapper\gradle-wrapper.properties`

استبدل محتوى الملف بالتالي:
```properties
#Gradle Wrapper Properties
#Updated to fix Flutter compatibility issues
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.7-all.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

### 2. تحديث settings.gradle (للمشاريع الحديثة)

**الملف:** `F:\idoomfiber_app\android\settings.gradle`

إذا كان مشروعك يستخدم الطريقة الحديثة (plugins block), استبدل المحتوى بالتالي:
```gradle
pluginManagement {
    def flutterSdkPath = {
        def properties = new Properties()
        file("local.properties").withInputStream { properties.load(it) }
        def flutterSdkPath = properties.getProperty("flutter.sdk")
        assert flutterSdkPath != null, "flutter.sdk not set in local.properties"
        return flutterSdkPath
    }()

    includeBuild("$flutterSdkPath/packages/flutter_tools/gradle")

    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }

    plugins {
        id "dev.flutter.flutter-plugin-loader" version "1.0.0"
        id "com.android.application" version "8.6.0" apply false
        id "org.jetbrains.kotlin.android" version "1.9.0" apply false
    }
}

plugins {
    id "dev.flutter.flutter-plugin-loader"
}

include ":app"
```

### 3. تحديث build.gradle الرئيسي (للمشاريع القديمة)

**الملف:** `F:\idoomfiber_app\android\build.gradle`

إذا كان مشروعك يستخدم الطريقة القديمة (buildscript), استبدل المحتوى بالتالي:
```gradle
buildscript {
    ext.kotlin_version = '1.9.0'
    repositories {
        google()
        mavenCentral()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:8.6.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.buildDir = '../build'
subprojects {
    project.buildDir = "${rootProject.buildDir}/${project.name}"
}
subprojects {
    project.evaluationDependsOn(':app')
}

tasks.register("clean", Delete) {
    delete rootProject.buildDir
}
```

### 4. تحديث build.gradle الخاص بالتطبيق

**الملف:** `F:\idoomfiber_app\android\app\build.gradle`

تأكد من أن الملف يحتوي على:
```gradle
plugins {
    id "com.android.application"
    id "kotlin-android"
    // The Flutter Gradle Plugin must be applied after the Android and Kotlin Gradle plugins.
    id "dev.flutter.flutter-gradle-plugin"
}

android {
    namespace = "com.example.idoomfiber_app"  // غير هذا إلى package name الخاص بك
    compileSdk = flutter.compileSdkVersion
    ndkVersion = flutter.ndkVersion

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = JavaVersion.VERSION_1_8
    }

    defaultConfig {
        applicationId = "com.example.idoomfiber_app"  // غير هذا إلى application ID الخاص بك
        minSdk = flutter.minSdkVersion
        targetSdk = flutter.targetSdkVersion
        versionCode = flutter.versionCode
        versionName = flutter.versionName
    }

    buildTypes {
        release {
            signingConfig = signingConfigs.debug
        }
    }
}

flutter {
    source = "../.."
}

dependencies {
    // Add any additional dependencies here
}
```

### 5. حل مشكلة CMake

إذا استمرت مشكلة CMake، جرب إحدى الحلول التالية:

**الحل الأول:** تحديث CMake
- افتح Android Studio
- اذهب إلى SDK Manager
- في تبويب SDK Tools، تأكد من تحديث CMake إلى أحدث إصدار

**الحل الثاني:** تعطيل CMake مؤقتاً
إذا كنت لا تستخدم C/C++ code، أضف هذا إلى `android/app/build.gradle`:
```gradle
android {
    // ... existing configuration
    
    packagingOptions {
        pickFirst '**/libc++_shared.so'
        pickFirst '**/libjsc.so'
    }
}
```

### 6. تنظيف وإعادة بناء المشروع

بعد تطبيق جميع التغييرات، افتح Command Prompt في مجلد المشروع الرئيسي ونفذ:

```bash
cd F:\idoomfiber_app
flutter clean
flutter pub get
flutter build apk
```

## ملاحظات مهمة:

1. **تأكد من تحديث package name:** غير `com.example.idoomfiber_app` إلى package name الفعلي لتطبيقك
2. **احتفظ بنسخة احتياطية:** قم بعمل backup للملفات قبل التعديل
3. **إعادة التشغيل:** قد تحتاج إلى إعادة تشغيل Android Studio بعد التغييرات
4. **فحص التوافق:** تأكد من أن جميع plugins في pubspec.yaml متوافقة مع الإصدارات الجديدة

## في حالة استمرار المشاكل:

1. احذف مجلد `build` في المشروع
2. احذف مجلد `.gradle` في مجلد android
3. نفذ `flutter clean` مرة أخرى
4. جرب البناء مع flag إضافي: `flutter build apk --verbose`

هذه الخطوات ستحل جميع المشاكل المذكورة في رسالة الخطأ.
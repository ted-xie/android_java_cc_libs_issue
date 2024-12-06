load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@rules_java//java:java_library.bzl", "java_library")
load("@rules_android//rules:rules.bzl", "android_library", "android_binary")

genrule(
  name = "foo_cc",
  outs = ["foo.cc"],
  cmd = "echo 'int foo();' > $@",
)

cc_library(
  name = "foo_lib",
  srcs = [":foo_cc"],
)

genrule(
  name = "Bar_java",
  outs = ["Bar.java"],
  cmd = "echo 'package com.example; public class Bar {}' > $@",
)

java_library(
  name = "Bar_lib",
  srcs = [":Bar_java"],
  deps = [":foo_lib"],
)

genrule(
  name = "Baz_java",
  outs = ["Baz.java"],
  cmd = "echo 'package com.example.android; public class Baz {}' > $@",
)

genrule(
  name = "AndroidManifest_xml",
  outs = ["AndroidManifest.xml"],
  cmd = """cat > $@ << EOT
<?xml version = "1.0" encoding = "utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android" >

    <uses-sdk
        android:minSdkVersion="21"
        android:targetSdkVersion="30" />

    <application>
        <activity
            android:name="com.example.android.Baz"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
EOT""",
)

android_library(
  name = "Baz_lib",
  srcs = [":Baz_java"],
  deps = [":Bar_lib"],
  manifest = ":AndroidManifest.xml",
)

android_binary(
  name = "Baz_app",
  deps = [":Baz_lib"],
  manifest = ":AndroidManifest.xml",
)

genrule(
  name = "so_file_from_app",
  outs = ["lib_from_app.so"],
  srcs = [":Baz_app.apk"],
  cmd = "set -e; unzip -q $(SRCS); find lib -name \"*.so\" -exec cp {} $@ \\;",
)

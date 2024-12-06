# cc deps to `java_library` to `android_binary` doesn't work

## TL;DR
A dependence chain of `cc_library -> java_library -> android_library -> android_binary` no longer materializes a .so files in the app's APK.

## Root cause
`rules_android` PR #291 removes `--experimental_google_legacy_api` from the .bazelrc file of `rules_android` itself. This flag guards the availability of two features used by `rules_android`:

* `java_common.add_constraints`: Adds `constraints = ["android"]` to all `android_library` and `android_binary` rules
* `JavaInfo.cc_link_params_info`: Returns the list of CcLinkParamsInfo providers from a JavaInfo provider.

The `java_common` attribute above should be an innocuous change for OSS users: the feature is not pertinent outside of Google-internal app builds. However, the latter attribute (`cc_link_params_info`)
may have pertinence for open source builds, if the dependence chain described in the "TL;DR" section exists. PR #291 refactors the Android rules to _optionally_ extract `JavaInfo.cc_link_params_info`
using `getattr()`. Because that attribute is not available unless `--experimental_google_legacy_api` is passed to Bazel, the CcLinkParamsInfo providers are no longer visible.

The net effect of this is that if a user's build depends on the `android_binary` rule to compile and link C++ dependencies, and to store the compiled C++ .so file into their APK, their library will no longer
appear as expected in the app.

## Reproducing the failure
With Bazel 8.0.0rc7 or 7.4.1: `bazel build //:so_file_from_app`. This target attempts to unzip the .so file from the Baz.apk file in `//:Baz_app`. A failure message should appear:

```
ERROR: /usr/local/google/home/tedx/work/link_params/BUILD:73:8: declared output 'lib_from_app.so' was not created by genrule.
```

## Reproducing the old successful behavior
First set up a git clone of `rules_android` checked out to `8021eedc11c9961fabb24a1144dfc40ae2c7a327`. Then, with bazel 8.0.0rc7 or 7.4.1: `bazel build //:so_file_from_app --override_module=rules_android=/path/to/your/rules_android`.

```
INFO: Found 1 target...
Target //:so_file_from_app up-to-date:
  bazel-bin/lib_from_app.so
```


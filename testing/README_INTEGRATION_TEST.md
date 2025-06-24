# How to Validate Integration Test Tasks Locally

> This document is used to introduce how to run integration test tasks locally

## Android

### Local Verification

First, we need to build the artifacts of the Android subproject in the lynx project.

- Environment Setup

```bash
source tools/envsetup.sh
tools/hab sync . -f
```

- Building Artifacts

```bash
cd platform/android/
./gradlew assembleAllModulesRelease -Penable_trace=None
./gradlew assembleAllModulesDevRelease -Penable_trace=perfetto
./gradlew publishAllModules  -PpublishDevRelease=true -Pversion=0.0.1-alpha.1
./gradlew zipAllArtifacts -Pversion=0.0.1-alpha.1
```

After executing the above steps, an `artifacts-collection.zip` file will be generated in the `platform/android/build` directory.

Validating artifact availability using [integration-lynx-demo](https://github.com/lynx-family/integrating-lynx-demo-projects)

```bash
git clone git@github.com:lynx-family/integrating-lynx-demo-projects.git
cd integrating-lynx-demo-projects
# commit_id may vary across different branches. It can be obtained from the `Cherry Pick Commit To Build With Local AAR` step in the `android-integration-test` task within the `./github/workflows/ci.yml` file of the lynx repository
git fetch origin 4c977faf04eb295fa2818a183e9dc66c5a7f5455 

cd android/JavaEmptyProject/
# LYNX_PROJECT_ROOT_PATH is the absolute path of the local lynx repository
python3 shell/prepare_package_with_local_aar.py $LYNX_PROJECT_ROOT_PATH/platform/android/build/artifacts-collection.zip 0.0.1-alpha.1
# Publish lynx android SDK to local maven
./gradlew publishAllZips
./gradlew :app:assembleDebug
```

### Breaking Change

For break changes that require modifying the `integration-lynx-demo`, follow these steps:
- Switch `integration-lynx-demo` to the corresponding branch (repository branches are in one-to-one correspondence with the lynx repository branches).

- After modifying the code, perform local validation. Once validation passes, submit a PR to the corresponding branch and push for code integration.

- Modify the ref parameter in the `Download Integration Demo Source Code` step of the `android-integration-test` task in the lynx repository's `./github/workflows/ci.yml` file, replacing it with the Head Commit Id of the corresponding branch in the `integration-lynx-demo` repository after the PR is merged.
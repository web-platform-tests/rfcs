# RFC 199: Add Bazel configuration files

## Summary

This RFC proposes adding a BUILD.bazel file and a tests.bzl file to the Web Platform Tests
(WPT) GitHub repository.

## Details

To support build systems like Bazel, implementors such as Cloudflare are currently required
to maintain patches on top of the Web Platform Tests GitHub repository. These patches expose
file groups representing each subfolder of the repository.

Adding Bazel configuration files will ensure that all projects wishing to run WPT in their
Bazel environments can build it without issues and without any modifications.

### Changes to the project

To add support, the WPT repository needs two new files:

1. `BUILD.bazel`: This file defines each subdirectory of the WPT repository under Bazel's `filegroup`.

```
[filegroup(
    name = dir,
    srcs = glob(["{}/**/*".format(dir)]),
    visibility = ["//visibility:public"],
) for dir in directories]
```

2. `tests.bzl` This file exports each subdirectory under a dictionary, allowing implementors
to access specific files inside a directory at the macro level of the build step.

```
TEST_GROUPS = {
    name: glob([name + "/**/*"])
    for name in directories
}
```

### Maintenance Considerations

- The proposed Bazel files are dynamically created and should not require
ongoing maintenance unless the folder structure of the WPT repository changes.
- The WPT repository could optionally include a bazel.yml GitHub Action to
monitor the linting and formatting of Bazel files. While this would increase
maintenance costs slightly, it would help prevent invalid Bazel files.
- Cloudflare employees, including Yagiz Nizipli (@anonrig) and
James Snell (@jasnell), have offered to maintain these files.

## Conclusion

Adding Bazel configuration files to the WPT repository will significantly improve
the integration process for projects using Bazel, reducing the need for
custom patches and streamlining the build process.

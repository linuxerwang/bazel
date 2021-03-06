package(default_visibility = ["//scripts/packages:__subpackages__"])

filegroup(
    name = "srcs",
    srcs = glob(["**"]) + [
        "//scripts/packages/debian:srcs",
        "//scripts/packages/fedora:srcs",
    ],
    visibility = [
        "//scripts:__pkg__",
        "//scripts/packages:__subpackages__",
    ],
)

filegroup(
    name = "packages",
    srcs = select({
        "//src:windows": [],
        "//src:windows_msvc": [],
        "//src:darwin": [
            ":install.sh",
            ":generate-package-info",
        ],
        "//src:darwin_x86_64": [
            ":install.sh",
            ":generate-package-info",
        ],
        "//conditions:default": [
            ":install.sh",
            ":generate-package-info",
            "//:bazel-distfile",
            "//scripts/packages/debian:bazel-debian",
            "//scripts/packages/debian:bazel-debian-src",
        ],
    }),
)

sh_binary(
    name = "package-info-generator",
    srcs = ["package_info_generator.sh"],
)

genrule(
    name = "generate-package-info",
    outs = ["README.md"],
    cmd = "$(location :package-info-generator) $$(find . -name '*status*.txt') >$@",
    stamp = 1,
    tools = [":package-info-generator"],
)

genrule(
    name = "generate-launcher",
    srcs = [
        "template_bin.sh",
        ":README.md",
    ],
    outs = ["launcher_bin.sh"],
    cmd = """
        release_info="$$(cat $(location :README.md))"
        template="$$(cat $(location template_bin.sh))"
        echo "$${template//%release_info%/$${release_info}}" >$@
        """,
)

genrule(
    name = "rename-bazel-bin",
    srcs = ["//src:bazel"],
    outs = ["bazel-real"],
    cmd = "cp $< $@",
)

genrule(
    name = "rename-bazel-sh",
    srcs = ["bazel.sh"],
    outs = ["bazel"],
    cmd = "cp $< $@",
)

load(":self_extract_binary.bzl", "self_extract_binary")

self_extract_binary(
    name = "install.sh",
    empty_files = ["tools/defaults/BUILD"],
    flatten_resources = [
        ":bazel-real",
        ":bazel",
        "//scripts:bash_completion",
    ],
    launcher = ":launcher_bin.sh",
)

genrule(
    name = "embedded_label",
    srcs = [],
    outs = [
        "version.txt",
    ],
    cmd =
        "find -L . -name stable-status.txt -exec cat '{}' ';'" +
        " | grep EMBED_LABEL | cut -d ' ' -f 2-" +
        " > $@\n",
    stamp = 1,
)

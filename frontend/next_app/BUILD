load("@build_bazel_rules_nodejs//:index.bzl", "nodejs_binary")
load("@io_bazel_rules_docker//container:container.bzl", "container_image")

package(default_visibility = ["//visibility:public"])

NPM_PACKAGES = [
    "@frapp_next_modules//:node_modules",
]

filegroup(
    name = "srcs",
    srcs = glob(["pages/**"]) + glob([
        "components/**",
    ]) + glob([
        "util/**",
    ]) + glob([
        "public/**",
    ]),
)

nodejs_binary(
    name = "next",
    data = NPM_PACKAGES,
    entry_point = "@frapp_next_modules//:node_modules/next/dist/bin/next",
)

genrule(
    name = "prod_build",
    srcs = NPM_PACKAGES + [
        ":srcs",
        "next.config.js",
        "postcss.config.js",
        "tailwind.config.js",
    ],
    outs = [
        "prodBuild",
    ],
    cmd = " && ".join([
        "cp -a frontend/next_app/ ./",
        "$(location next) build",
        "mkdir prodBuild",
        "mv external/frapp_next_modules/node_modules nextBuild public/ next.config.js tailwind.config.js prodBuild",
        "mv prodBuild $@",
    ]),
    tools = [":next"],
)

container_image(
    name = "prod_image",
    base = "@node_base//image",
    cmd = "".join([
        "ls;",
        "chmod +x ./entry.sh;",
        "./entry.sh;",
        "cd prodBuild/;",
        "ls;",
        "node_modules/next/dist/bin/next start;",
    ]),
    files = [
        "entry.sh",
        ":prod_build",
    ],
)

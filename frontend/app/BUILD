load("@frapp_modules//http_server:index.bzl", "http_server")
load("@build_bazel_rules_nodejs//:index.bzl", "nodejs_binary")
load("@io_bazel_rules_docker//container:container.bzl", "container_image")
load("@rules_proto_grpc//js:defs.bzl", "js_grpc_web_library")

package(default_visibility = ["//visibility:public"])

NPM_PACKAGES = [
    "@frapp_modules//:node_modules",
]

js_grpc_web_library(
    name = "task_grpc_js",
    deps_repo = "@frapp_modules",
    protos = [
        "//proto/task:task_service_proto",
        "//proto/user:user_proto",
    ],
)

filegroup(
    name = "srcs",
    srcs = glob(
        ["src/**"],
        exclude = ["src/genProto/**"],
    ) + glob(["assets/**"]),
)

nodejs_binary(
    name = "webpack_cli",
    data = NPM_PACKAGES,
    entry_point = "@frapp_modules//:node_modules/webpack-cli/bin/cli.js",
)

genrule(
    name = "prod_build",
    srcs = NPM_PACKAGES + [
        ":srcs",
        "src/index.html",
        "webpack.prod.js",
        ".babelrc",
        ":task_grpc_js",
    ],
    outs = [
        "dist",
    ],
    cmd = " && ".join([
        "cp -a bazel-out/darwin-fastbuild/bin/frontend/app/task_grpc_js_pb/ frontend/app/src/",
        #"ls -R frontend | grep ':$$' | sed -e 's/:$$//' -e 's/[^-][^\/]*\//--/g' -e 's/^/   /' -e 's/-/|/'",
        "$(execpath webpack_cli) --mode=production --config=$(location webpack.prod.js)",
        "mv frontend/app/dist $@",
    ]),
    tools = [":webpack_cli"],
)

genrule(
    name = "dev_build",
    srcs = NPM_PACKAGES + [
        ":srcs",
        "src/index.html",
        "webpack.dev.js",
        ".babelrc",
        ":task_grpc_js",
    ],
    outs = [
        "devDist",
    ],
    cmd = " && ".join([
        "cp -a bazel-out/darwin-fastbuild/bin/frontend/app/task_grpc_js_pb/ frontend/app/src/",
        "$(execpath webpack_cli) --mode=development --config=$(location webpack.dev.js)",
        "mv frontend/app/devDist $@",
    ]),
    tools = [":webpack_cli"],
)

http_server(
    name = "prod_server",
    data = [
        ":prod_build",
    ],
    # args = ["frontend/app/dist/"],
)

http_server(
    name = "dev_server",
    args = ["frontend/app/devDist"],
    data = [
        ":dev_build",
    ],
)

container_image(
    name = "dev_image",
    base = "@nginx_base//image",
    cmd = "".join([
        "chmod +x ./entry.sh;",
        "./entry.sh;",
        "nginx -c /nginxDev.conf;",
    ]),
    files = [
        "entry.sh",
        "nginxDev.conf",
        ":dev_build",
    ],
)

container_image(
    name = "prod_image",
    base = "@nginx_base//image",
    cmd = "".join([
        "chmod +x ./entry.sh;",
        "./entry.sh;",
        "nginx -c /nginx.conf;",
    ]),
    files = [
        "entry.sh",
        "nginx.conf",
        ":prod_build",
    ],
)

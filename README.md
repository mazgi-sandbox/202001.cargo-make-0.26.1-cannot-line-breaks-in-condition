# 202001.cargo-make-0.26.1-cannot-line-breaks-in-condition

Does cargo-make cannot parse the Makefile if it included line breaks in a condition statement?

## How to Representation

### (prepare) Download latest cargo-make

Linux:

```shellsession
export CARGO_MAKE_VERSION="0.26.1" \
&& curl -sL https://github.com/sagiegurari/cargo-make/releases/download/${CARGO_MAKE_VERSION}/cargo-make-v${CARGO_MAKE_VERSION}-x86_64-unknown-linux-musl.zip \
| busybox unzip -p - cargo-make-v${CARGO_MAKE_VERSION}-x86_64-unknown-linux-musl/cargo-make > bin/cargo-make && chmod a+x bin/cargo-make
```

macOS:

```shellsession
export CARGO_MAKE_VERSION="0.26.1" \
&& curl -sL https://github.com/sagiegurari/cargo-make/releases/download/${CARGO_MAKE_VERSION}/cargo-make-v${CARGO_MAKE_VERSION}-x86_64-apple-darwin.zip \
| bsdtar --strip-components 1 -C bin/ -xvf - cargo-make-v${CARGO_MAKE_VERSION}-x86_64-apple-darwin/cargo-make
```

### Representation

Check cargo-make version.

```shellsession
❯ bin/cargo-make make -V
cargo-make 0.26.1
```

The case without line breaks in the condition statement.  
...succeed :blush:

```shellsession
❯ bin/cargo-make make --makefile without-line-breaks.toml
[cargo-make] INFO - cargo make 0.26.1
[cargo-make] INFO - Build File: without-line-breaks.toml
[cargo-make] INFO - Task: default
[cargo-make] INFO - Profile: development
[cargo-make] INFO - Running Task: empty
[cargo-make] INFO - Running Task: default
[cargo-make] INFO - Running Task: empty
[cargo-make] INFO - Build Done in 0 seconds.
```

```shellsession
❯ cat without-line-breaks.toml
[config]
min_version = "0.26.1"
skip_core_tasks = true

[tasks.default]
condition = { files_exist = ["${CARGO_MAKE_WORKING_DIRECTORY}/README.md"], files_not_exist = ["${CARGO_MAKE_WORKING_DIRECTORY}/not-exist"] }
```

The case **with line breaks** in the condition statement.  
...fail :fearful:

```shellsession
❯ bin/cargo-make make --makefile with-line-breaks.toml
[cargo-make] INFO - cargo make 0.26.1
thread 'main' panicked at 'Unable to parse external descriptor, expected a table key, found a newline at line 6 column 14', src/lib/descriptor.rs:380:27
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

```shellsession
❯ cat with-line-breaks.toml
[config]
min_version = "0.26.1"
skip_core_tasks = true

[tasks.default]
condition = {
  files_exist = ["${CARGO_MAKE_WORKING_DIRECTORY}/README.md"],
  files_not_exist = ["${CARGO_MAKE_WORKING_DIRECTORY}/not-exist"]
}
```

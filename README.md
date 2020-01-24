# 202001.cargo-make-0.26.1-cannot-line-breaks-in-condition

~~Does cargo-make cannot parse the Makefile if it included line breaks in a condition statement?~~

=> This behavior is correct that depends on TOML spec.

See also:

- https://github.com/sagiegurari/cargo-make/issues/372#issuecomment-578108906
- https://github.com/toml-lang/toml#keys

We can set conditions as multiple lines with "dotted keys" [like this](https://github.com/mazgi-sandbox/202001.cargo-make-0.26.1-cannot-line-breaks-in-condition/blob/523e63582b110fe68d309f38985228691ea6b5ba/respect-toml-spec.toml#L10-L11).

```toml
condition.foo = [...]
condition.bar = [...]
```

---

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

~~The case **with line breaks** in the condition statement.~~  
~~...fail :fearful:~~

=> This behavior is correct!

```shellsession
❯ bin/cargo-make make --makefile with-line-breaks.toml
[cargo-make] INFO - cargo make 0.26.1
thread 'main' panicked at 'Unable to parse external descriptor, expected a table key, found a newline at line 6 column 14', src/lib/descriptor.rs:380:27
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

<details>
<summary>

```shellsession
❯ RUST_BACKTRACE=full bin/cargo-make make --makefile with-line-breaks.toml
```

</summary>

```shellsession
❯ RUST_BACKTRACE=full bin/cargo-make make --makefile with-line-breaks.toml
[cargo-make] INFO - cargo make 0.26.1
thread 'main' panicked at 'Unable to parse external descriptor, expected a table key, found a newline at line 6 column 14', src/lib/descriptor.rs:380:27
stack backtrace:
   0:           0x786104 - backtrace::backtrace::libunwind::trace::h0d3b97a6b64193be
                               at /cargo/registry/src/github.com-1ecc6299db9ec823/backtrace-0.3.40/src/backtrace/libunwind.rs:88
   1:           0x786104 - backtrace::backtrace::trace_unsynchronized::hb78a971d8547111e
                               at /cargo/registry/src/github.com-1ecc6299db9ec823/backtrace-0.3.40/src/backtrace/mod.rs:66
   2:           0x786104 - std::sys_common::backtrace::_print_fmt::ha1f41287e6ef2374
                               at src/libstd/sys_common/backtrace.rs:77
   3:           0x786104 - <std::sys_common::backtrace::_print::DisplayBacktrace as core::fmt::Display>::fmt::h1130808f9197ccb5
                               at src/libstd/sys_common/backtrace.rs:61
   4:           0x7bf2ac - core::fmt::write::h4fa8b44d73117031
                               at src/libcore/fmt/mod.rs:1028
   5:           0x7820f7 - std::io::Write::write_fmt::hc08a67176e074177
                               at src/libstd/io/mod.rs:1412
   6:           0x788c6e - std::sys_common::backtrace::_print::h33a300037b4dc1ed
                               at src/libstd/sys_common/backtrace.rs:65
   7:           0x788c6e - std::sys_common::backtrace::print::h2ceebd24f74a6808
                               at src/libstd/sys_common/backtrace.rs:50
   8:           0x788c6e - std::panicking::default_hook::{{closure}}::h599b8688602f475a
                               at src/libstd/panicking.rs:188
   9:           0x788961 - std::panicking::default_hook::h0085905ac97017e0
                               at src/libstd/panicking.rs:205
  10:           0x78936b - std::panicking::rust_panic_with_hook::h780f8e9bbe4fc091
                               at src/libstd/panicking.rs:464
  11:           0x788f0e - std::panicking::continue_panic_fmt::h1444a364e5f24a1a
                               at src/libstd/panicking.rs:373
  12:           0x788e4f - std::panicking::begin_panic_fmt::h35a34eca8fc5e443
                               at src/libstd/panicking.rs:328
  13:           0x537979 - cli::descriptor::load_external_descriptor::hba43f1dfe27742e9
  14:           0x538f50 - cli::descriptor::load_descriptors::h1e9a6d0f1e1f42cf
  15:           0x53af8b - cli::descriptor::load::h8d00c32f86ea33ff
  16:           0x41ad64 - cli::cli::run_cli::h20ba08cbc590f3f0
  17:           0x40c9ec - cli::run_cli::h532a9bcfa4174ef8
  18:           0x4004af - cargo_make::main::h7469f827b703c138
  19:           0x400683 - std::rt::lang_start::{{closure}}::h96deade27f6197b0
  20:           0x788d93 - std::rt::lang_start_internal::{{closure}}::hdb12a3ead60a8960
                               at src/libstd/rt.rs:48
  21:           0x788d93 - std::panicking::try::do_call::h75828266d21559c3
                               at src/libstd/panicking.rs:287
  22:           0x79100a - __rust_maybe_catch_panic
                               at src/libpanic_unwind/lib.rs:78
  23:           0x7896eb - std::panicking::try::h2533a62d10a8f001
                               at src/libstd/panicking.rs:265
  24:           0x7896eb - std::panic::catch_unwind::hd9ecf47e7cd9602d
                               at src/libstd/panic.rs:396
  25:           0x7896eb - std::rt::lang_start_internal::h93ee4050c8419278
                               at src/libstd/rt.rs:47
  26:           0x400502 - main
```

</details>

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

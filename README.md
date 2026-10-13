# rocq2md

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A command-line utility written in Rust that converts Rocq files (`.v`) to Markdown (`.md`).

## Description

`rocq2md` converts Rocq files to Markdown by transforming block comments to text and Rocq code to Markdown code blocks.

## Installation

### Prerequisites

- [Rust](https://www.rust-lang.org/tools/install) 1.70.0 or later.
- Cargo (comes with Rust).

### Clone the repo and build the package

```bash
git clone https://github.com/matematiflo/rocq2md.git
cd rocq2md
cargo build
```

## Running the program

The program can be run directly after cloning (no build necessary) or as a local or global executable.

### Directly after cloning

The program can be run directly using `cargo run -- <arguments>`. A test file `sample.v` is provided.

```bash
cargo run -- test_files/sample.v
```

### Local usage (inside the repo)

After building an executable, you can run it as follows. Use either

```bash
cargo build
./target/debug/rocq2md <arguments> 
```

or (for an optimized executable)

```bash
cargo build --release
./target/release/rocq2md <arguments>
```

### Adding the executable to your Rust environment

If you want to add `rocq2md` to your Rust environment, install the package globally as follows.

```bash
cargo install --path .
```

Check installation:

```bash
rocq2md --version
```

Check location (should be `~/.cargo/bin`):

```bash
which rocq2md
```

Alternatively, you can [use cargo](#rust-executable) to install the package directly from [crates.io](https://crates.io/crates/rocq2md).

## Commands

`rocq2md` can convert single files or all Rocq files in a specified source directory (including its subfolders). For single files, you can specify the target (location and name). For directories, you can specify the target directory (which is created automatically if not already present).

```bash
rocq2md <file.v>                # converts <file.v> to <file.md>
rocq2md <src_dir>               # converts all Rocq files in <src_dir> to Markdown files
rocq2md <file.v> <new_name.md>  # converts <file.v> to <new_name.md> (subfolders allowed)
rocq2md <src_dir> <tgt_dir>     # converts all Rocq files from <src_dir>; puts them in <tgt_dir>
```

Examples you can run in this repo:

```bash
rocq2md test_files/sample.v
rocq2md test_files/sample.v converted_files/sample.md
rocq2md test_files
rocq2md test_files converted_files
```

## Features

- Converts top-level Rocq comments `(* ... *)` to Markdown text.
- Puts Rocq code inside ` ```rocq ` code fences.
- Preserves directory structure from source to target.
- Nested folders are also processed.
- Any text enclosed between `--#--` symbols is removed from the generated markdown file.
- A `(* ... *)` comment is converted to Markdown only if the `(*` appears strictly at the beginning of a line (column 0) and the comment is not contained inside a proof environment.
- In nested comments `(* (* ... *) ... *)`, only the outermost delimiters are stripped.
- Comments that are not converted generate a warning during conversion. The reason for the warning is logged into the `rocq2md.log` file (in the absence of any warning, no log file is created).
- Comments contained in a proof (are not converted and) do not generate a warning. The total number of warnings is printed to the console after conversion.

## Basic usage

### Test file

The test file `sample.v`

````rocq
(* # Factorial

Let us introduce the factorial function. --#-- side comment, removed from comversion ---#-- It is defined recursively. *)

Fixpoint fact (n : nat) := (* not converted: inline trailing comment *)
  match n with
  | 0   => 1
  | S n' => n * fact n'  (* note the role of `n'` here --#-- extra note: this comment should be removed --#-- *)
  end.

  (* not converted: does not start at columnn 0 *)

Compute fact 5.

Lemma fact_succ (n' : nat) : fact (S n') = S n' * fact n'.
Proof.
  reflexivity.
  (* the proof holds by reflexivity --#-- this comment is inside a proof and is not converted to text --#-- *)
Qed.
````

is converted to

````markdown
# Factorial

Let us introduce the factorial function. It is defined recursively.

```rocq
Fixpoint fact (n : nat) := (* not converted: inline trailing comment *)
  match n with
  | 0   => 1
  | S n' => n * fact n'  (* note the role of `n'` here *)
  end.

  (* not converted: does not start at columnn 0 *)

Compute fact 5.

Lemma fact_succ (n' : nat) : fact (S n') = S n' * fact n'.
Proof.
  reflexivity.
  (* the proof holds by reflexivity *)
Qed.
```
````

and produces the following message.

```bash
Converting test_files/sample.v to test_files/sample.md
3 warning(s): 3 unconverted comment(s) and 0 unclosed comment(s); see rocq2md.log for details.
Note: 3 block(s) ignored due to --#-- markers.
```

### Ignored text

Text placed between two `--#--` markers inside a converted comment block is removed from the Markdown output, along with the markers themselves. This works both inline (on a single line) and as a block spanning multiple lines:

```rocq
(* Keep this. --#-- Remove this internal note. --#-- Keep this too.

--#--
This whole paragraph is removed as well.
--#--

Kept again. *)
```

## Package information

### Files

- `src/rocq2md_core.rs`: Parsing and converting rules.
- `src/lib.rs`: Library interface that exports public functions.
- `src/main.rs`: Command-line interface.

### Rust executable

You can install `rocq2md` from [crates.io](https://crates.io/crates/rocq2md):

```bash
cargo install rocq2md
```

This will install the package in your `~/.cargo/bin` directory, making it available in your PATH.

Alternatively, you can [install your locally built version](#adding-the-executable-to-your-rust-environment):

```bash
cargo install --path .
```

### Testing

Unit tests are located in `src/rocq2md_core.rs` and cover comment conversion, column-0 detection, proof-environment detection, nested comments, and the `--#--` marker.

```bash
# Run all tests
cargo test
```

### Acknowledgments

- [arthurpaulino/lean2md](https://github.com/arthurpaulino/lean2md), the original Lean to Markdown conversion tool, written in Python.
- [Seasawher/mdgen](https://github.com/Seasawher/mdgen), based on `lean2md` and written in Lean (to generate documentation directly inside a Lean project).
- [fosterfarrell9/lean2md](https://github.com/fosterfarrell9/lean2md), based on `mdgen` and written in Rust.

### License

This software is published under an [MIT License](https://opensource.org/licenses/MIT).

### Contributing

To contribute, open an issue or submit a pull request.

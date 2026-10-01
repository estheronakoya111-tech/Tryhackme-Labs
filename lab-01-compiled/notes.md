
# 🧩 TryHackMe: Compiled — Reverse Engineering Writeup

## 📌 Room Overview
* **Target:** Linux ELF Binary (`Compiled`)
* **Category:** Reverse Engineering / Dynamic Binary Analysis
* **Difficulty:** Easy
* **Core Vulnerability:** Hardcoded Validation Logic & Reliance on Unstripped Standard Library Symbols

---

## 1. 🔍 Initial Reconnaissance & Setup

After downloading the binary file, initial execution failed due to missing execution permissions and a lengthy default filename.

### Step 1: File Preparation & Permissions
Before inspecting the binary, I cleaned up the filename and added execution privileges:

```bash
# Rename the downloaded binary to a cleaner name
mv Compiled-1688545393558.Compiled Compiled

# Grant executable permissions to the ELF file
chmod +x Compiled

```

#### Command Breakdown:

* **`mv`**: Moves or renames files in Linux.
* **`chmod +x`**: Adds execution permissions (`+x`) so the operating system allows running the ELF binary as an executable program.

---

## 2. 🔬 Binary Analysis & Disassembly

Reverse engineering the binary required analyzing both its static memory strings and its dynamic execution flow.

### Step 1: Format String Identification

Analyzing the binary's read-only data section (`.rodata`) revealed format specifiers used by standard input functions:

```text
Password: 
DoYouEven%sCTF

```

The format string `DoYouEven%sCTF` indicates that `scanf()` expects an input starting with `DoYouEven` and ending with `CTF`, capturing the middle string passed into `%s`.

---

### Step 2: Dynamic Library Tracing (`ltrace`)

To inspect function calls made by the program at runtime without needing a heavy decompiler GUI, I executed the binary under `ltrace`:

```bash
ltrace ./Compiled

```

#### What is `ltrace`?

`ltrace` (Library Trace) is a debugging utility that intercepts and records dynamic library calls (such as `scanf`, `strcmp`, and `printf`) made by an executed process.

#### Execution Trace Output:

```text
fwrite("Password: ", 1, 10, 0x7f4af7c6f5c0)    = 10
__isoc99_scanf("DoYouEven%sCTF", "DoYouEven_init") = 1
strcmp("_init", "__dso_handle")                  = 10
strcmp("_init", "_init")                        = 0
printf("Correct!")                              = 8

```

---

## 3. ⚡ Exploit & Password Reconstruction

### How the Validation Logic Works

1. **Input Parsing:** `__isoc99_scanf` parses the user input against `DoYouEven%sCTF`, stripping the `DoYouEven` prefix and extracting the inner string parameter.
2. **First Check:** `strcmp` compares the extracted parameter against `__dso_handle` (returns non-zero / mismatch).
3. **Second Check:** `strcmp` compares the extracted parameter against `_init`.
4. **Validation:** `strcmp("_init", "_init")` evaluates to `0` (exact match), triggering the `printf("Correct!")` code block.

### Final Password

Combining the required prefix wrapper (`DoYouEven`) with the target string (`_init`) yields the full flag/password:

**`DoYouEven_init`**

---

## 4. 🛡️ Remediation & Security Best Practices

1. **Avoid Hardcoded Logic:** Secrets, key validation parameters, and comparison targets should never be stored directly or matched in plaintext within binary memory.
2. **Strip Symbol Tables:** Production binaries should be compiled with symbol stripping (`strip --strip-all` or `gcc -s`) to prevent attackers from tracing internal function parameters and variable names using utilities like `ltrace`.

```


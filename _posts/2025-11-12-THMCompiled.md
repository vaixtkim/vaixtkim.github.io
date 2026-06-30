---
title: TryHackMe - Compiled
categories: [TryHackMe, Reverse Engineering]
tags: [tryhackme, reverse-engineering, compiled]     # TAG names should always be lowercase
author: vaixtkim
toc: false
---

# TryHackMe - Compiled

**Link:** https://tryhackme.com/room/compiled

## Description

This is a reverse engineering challenge from TryHackMe that requires analyzing a compiled binary to discover the correct password.

- **Challenge category:** Reverse Engineering
- **Difficulty level:** Easy
- **Platform:** TryHackMe
- **Files provided:** `Compiled.Compiled` (binary executable)
- **Challenge prompt:** Download the task file and find the password. The binary can also be found in the AttackBox inside the `/root/Rooms/Compiled/` directory.

**Note:** The binary will not execute if using the AttackBox. However, you can still solve the challenge through static analysis.

**Question to answer:** What is the password?

## Solution

### Initial Analysis and Observations

I downloaded the provided binary file and attempted to execute it:

```bash
└─$ ./Compiled.Compiled
Password: test
Try again!
```

The binary prompts for a password and rejects incorrect inputs. Since we need to find the correct password, reverse engineering the binary through static analysis is necessary.

### Tools Used

- **Ghidra** - Free and open-source reverse engineering tool for decompiling the binary

### Step-by-Step Approach

#### 1. Decompiling the Binary

I loaded the binary into Ghidra and analyzed the `main` function. Here's the decompiled code:

```c
undefined8 main(void)
{
  int iVar1;
  char local_28 [32];
  
  fwrite("Password: ",1,10,stdout);
  __isoc99_scanf("DoYouEven%sCTF",local_28);
  iVar1 = strcmp(local_28,"__dso_handle");
  if ((-1 < iVar1) && (iVar1 = strcmp(local_28,"__dso_handle"), iVar1 < 1)) {
    printf("Try again!");
    return 0;
  }
  iVar1 = strcmp(local_28,"_init");
  if (iVar1 == 0) {
    printf("Correct!");
  }
  else {
    printf("Try again!");
  }
  return 0;
}
```

#### 2. Code Analysis

Let's break down what this code does:

1. **Line 6:** Displays "Password: " prompt to the user
2. **Line 7:** Receives input using `scanf` with format string `"DoYouEven%sCTF"` and stores the captured portion in `local_28`
3. **Lines 8-12:** Compares `local_28` with `"__dso_handle"` - if equal, prints "Try again!" and exits
4. **Lines 13-19:** Compares `local_28` with `"_init"` - if equal, prints "Correct!"

### Key Insights or Techniques

The critical insight is understanding how the `scanf` format string `"DoYouEven%sCTF"` works:

- It expects input to start with the literal string "DoYouEven"
- The `%s` captures everything after "DoYouEven" until whitespace is encountered
- The captured string is stored in `local_28`

**Examples of how scanf captures input:**

| Input                        | Captured in `local_28` |
|------------------------------|------------------------|
| `DoYouEven_CTF tttt`        | `_CTF`                 |
| `DoYouEvenanytext`          | `anytext`              |
| `DoYouEven-CTF t`           | `-CTF`                 |

#### 3. Finding the Solution

To make the program print "Correct!", we need `local_28` to equal `"_init"`.

Since the scanf captures everything between "DoYouEven" and whitespace into `local_28`, the password should be:

**`DoYouEven_init`**

### Final Exploit or Solution Method

Testing the password:

```bash
└─$ ./Compiled.Compiled 
Password: DoYouEven_init
Correct!
```

Success! The password is accepted.

### Flag
<details>
<summary>Spoiler</summary>

  ``` 
  DoYouEven_init
  ```
</details>


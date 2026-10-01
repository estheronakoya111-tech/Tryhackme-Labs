


# My W1seGuy Challenge Notes — TryHackMe

## Overview

These are my personal writeup notes for the **W1seGuy** CTF challenge on TryHackMe. In this room, I performed a **Known-Plaintext Attack (KPA)** against a target server that encrypts data using a repeating 5-character XOR cipher over a raw TCP port.

---

## Technical Concepts & Tools I Used

* **Protocol:** TCP (Transmission Control Protocol) on Port 1337
* **Cipher:** Single-key repeating XOR ($\oplus$)
* **Key Length:** 5 characters (alphanumeric)
* **Tools Used:**
  * `netcat` (`nc`): Terminal tool I used to initially connect to the raw TCP port.
  * `python3`: Python socket script I wrote to calculate the XOR key and automate the response in real time.

---

## Step-by-Step Walkthrough

### Step 1: Connecting with Netcat

First, I tested the connection using Netcat:

```bash
nc <TARGET_IP> 1337

```

The server gave me a hex string and prompted me for the encryption key:

```text
This XOR encoded text has flag 1: <HEX_STRING>
What is the encryption key?

```

---

### Step 2: Understanding the Cryptography

Since XOR ($\oplus$) is symmetric:

$$\text{Ciphertext} = \text{Plaintext} \oplus \text{Key}$$

I reversed the equation to recover the key using the known plaintext:

$$\text{Key} = \text{Ciphertext} \oplus \text{Plaintext}$$

I relied on two known properties of TryHackMe flag formats:

1. Every flag starts with `THM{` (4 characters).
2. Every flag ends with `}` (1 character).

Since the key is 5 characters long and the message length is 40 bytes ($40 \pmod 5 = 0$), I mapped each position:

* Byte 0 $\oplus$ `'T'` = Key Character 1
* Byte 1 $\oplus$ `'H'` = Key Character 2
* Byte 2 $\oplus$ `'M'` = Key Character 3
* Byte 3 $\oplus$ `'{'` = Key Character 4
* Byte 39 $\oplus$ `'}'` = Key Character 5

---

### Step 3: Writing My Python Solver Script

Because the server generates a new key every time a connection opens, I wrote a Python script with the help of Gemini to handle the TCP connection, derive the key, and send it back before the session timed out.

Here is the script I saved as `solve.py`:

```python
import socket

TARGET_IP = "10.10.x.x"  # Replace with active target IP
TARGET_PORT = 1337

def solve():
    # Step A: Connect to TCP socket
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((TARGET_IP, TARGET_PORT))

    # Step B: Read server response containing the hex string
    response = s.recv(1024).decode()
    hex_str = response.split("flag 1: ")[1].split("\n")[0].strip()
    ciphertext = bytes.fromhex(hex_str)

    # Step C: Derive the 5-character key using known plaintext
    key_bytes = [
        ciphertext[0] ^ ord('T'),
        ciphertext[1] ^ ord('H'),
        ciphertext[2] ^ ord('M'),
        ciphertext[3] ^ ord('{'),
        ciphertext[39] ^ ord('}')
    ]
    key = bytes(key_bytes)

    # Step D: Decrypt Flag 1
    flag1 = bytes([b ^ key[i % 5] for i, b in enumerate(ciphertext)]).decode()

    # Step E: Submit key back to receive Flag 2
    s.sendall(key + b"\n")
    flag2_response = s.recv(1024).decode().strip()

    s.close()

    print(f"[+] Flag 1: {flag1}")
    print(f"[+] Encryption Key: {key.decode()}")
    print(f"[+] Server Response (Flag 2): {flag2_response}")

if __name__ == "__main__":
    solve()

```

---

### Step 4: Running the Script

I executed the script in my terminal:

```bash
python3 solve.py

```

Output received:

```text
[+] Flag 1: THM{p1alnTextAtt4ckcAnr3alLyhUrty0urxOr}
[+] Encryption Key: Jg06t
[+] Server Response (Flag 2): Congrats! That is the correct key! Here is flag 2: THM{Brut3_ForC1nG_XOR_cAn_B3_FuN_n0?}

```

---

## Flags Recovered

| Flag | Value |
| --- | --- |
| **Flag 1** | `THM{p1alnTextAtt4ckcAnr3alLyhUrty0urxOr}` |
| **Flag 2** | `THM{Brut3_ForC1nG_XOR_cAn_B3_FuN_n0?}` |

---

## What I Learned

1. **Reusing XOR keys is dangerous:** Reusing a static or short key across a stream cipher makes it easy to recover the key using known-plaintext structures.
2. **Standard cryptography matters:** Custom XOR implementations should be replaced with robust, authenticated ciphers like AES-GCM or ChaCha20-Poly1305.

```


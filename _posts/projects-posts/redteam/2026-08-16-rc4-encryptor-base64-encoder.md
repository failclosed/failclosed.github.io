---
layout: post
title: RC4 Encryptor and Base64 Encoder
gh-repo: 4D5A/Red-Team-Projects
gh-badge: [follow, star]
categories: [redteam, projects]
tags: [PowerShell, Cryptography, infosec, Detection Engineering]
---

This is a small PowerShell project that RC4 encrypts a file, base64 encodes the result, and can optionally push the base64 through a swapped alphabet on the way out. Run it in reverse with the same key and you get the original file back. The RC4 function comes from Remko Weijnen and the alphabet translation from Doug Finke, both used with permission.

It's old, it's simple, and it does exactly what the name says. I'm writing about it as a Red Team piece for two reasons, and neither of them is that you should reach for it on an engagement. RC4 plus base64 plus PowerShell is the canonical shape of a loader, so a benign implementation makes a clean specimen for talking about why that shape gets caught. It's also a tidy example of the difference between obfuscation and encryption, which matters more on the offensive side than people new to it tend to assume.

## Defender quarantines it on sight

Before getting to any of the cryptography, the interesting behavior showed up on the first run. On an up to date Windows 10 box with Microsoft Defender at its defaults, the script wouldn't execute:

```
& : Operation did not complete successfully because the file
    contains a virus or potentially unwanted software.
```

The detection wasn't generic:

```
Threat   : TrojanDropper:PowerShell/Cobacis.B
Resource : ...\rc4-b64.ps1
```

To run it at all I had to add an exclusion for the directory.

A roughly decade old educational script, openly credited to two named authors, trips a Cobalt Strike adjacent dropper signature. That's not a false positive in the usual sense. Defender is matching the structure of the code, and the structure of the code is genuinely the structure of a stager: a byte array key, a 256 iteration key schedule, an XOR keystream loop, and base64 on the output. Nothing about the signature cares that the intent here is a demonstration.

As far as detection is concerned, the pattern is the payload.

## What it actually does

With the exclusion in place it runs fine. Here it is against a file containing the word `test`, walked through every stage:

<img src="{{ 'assets/img/2026-08-16-rc4-encryptor-base64-encoder/rc4-demo-run.png' | relative_url }}" alt="PowerShell session running rc4-b64.ps1 on the word test, showing the RC4 ciphertext bytes DC 9E 33 50, the base64 string 3J4zUA==, and a successful round-trip back to test" />

The pipeline is:

1. Read the file into a byte array.
2. RC4 encrypt it with a key, hardcoded in the script as the string `pass code`.
3. Base64 encode the encrypted bytes.
4. Optionally run the base64 text through a substitution that swaps the standard alphabet for a reordered one.

Each stage is written to its own file on disk, so you can inspect the whole transformation byte by byte:

<img src="{{ 'assets/img/2026-08-16-rc4-encryptor-base64-encoder/rc4-output-files.png' | relative_url }}" alt="Contents of the files written by the script: originalfile.txt reads test, EncryptedBytes.txt holds the RC4 bytes 220 158 51 80, b64EncodedEncryptedString.txt holds 3J4zUA==, and DecryptedString.txt reads test again" />

The RC4 itself is textbook. A key schedule permutes a 256 byte state array, then the generator walks that state producing a keystream that's XORed against the data:

```powershell
$i = $j = 0
for ($x = 0; $x -lt $buffer.Length; $x++) {
    $i = ($i + 1) % 256
    $j = ($j + $s[$i]) % 256
    $temp = $s[$i]; $s[$i] = $s[$j]; $s[$j] = $temp
    [int]$t = ($s[$i] + $s[$j]) % 256
    $buffer[$x] = $buffer[$x] -bxor $s[$t]
}
```

The optional alphabet swap maps `A-Za-z0-9+/` onto a reordered version of itself. Turning it on for the same input gives `T9UPk0==` instead of `3J4zUA==`. The underlying ciphertext bytes are identical. All that changed is which base64 characters represent them.

## Obfuscation isn't encryption

It's easy to look at "RC4 encrypted and base64 encoded with a custom alphabet" and read it as three layers of protection. It's one layer, and a weak one, wrapped in two layers of reversible encoding.

**The custom alphabet adds nothing.** A one to one remapping of the base64 characters is a monoalphabetic substitution. There's no key. Anyone who notices the output is base64 shaped can recover the standard alphabet by frequency or by trying the obvious mappings, and the substitution table is sitting in plaintext in the script anyway.

**Base64 isn't encryption at all.** It's an encoding, reversible by definition and by design. That's obvious when you state it plainly, and it's still one of the most common things I see mislabelled.

**The RC4 layer is the only cryptography here, and it's used in the weakest possible way.** Two problems compound.

The key is hardcoded, so every copy of the script encrypts with the string `pass code`. The key isn't secret from anyone who has the script, which is anyone you sent an encrypted file to.

Worse, the key is fixed with no IV or nonce, so every file is encrypted under the identical keystream. RC4 is a stream cipher, and reusing a keystream across two messages is the many time pad failure. It's catastrophic rather than gradual. XOR two ciphertexts together and the keystream cancels out, leaving you with the XOR of the two plaintexts. Recover or guess one and you get the other for free. I confirmed this against the script's own construction:

```
C1 xor C2 == P1 xor P2      : True
given P1 = "ATTACK AT DAWN"
recovered P2                : RETREAT NOW!!!
```

No key was needed to pull the second message out. That isn't an implementation bug in this script specifically, it's what happens to any stream cipher used with a static key and no per message randomness.

RC4 on top of that is simply retired. It was prohibited in TLS by RFC 7465 in 2015 over its keystream biases, the same statistical leaks that made the early WEP attacks work. It shouldn't be carrying anything you actually need kept secret in 2026.

None of this makes the script bad at what it is, which is a demonstration. It makes it a good illustration of a trap: layering reversible encodings around one weak cipher and mistaking the total for strength.

## Detection

The useful question isn't how to catch this exact script, which Defender already does by name. It's how to catch the pattern once the constants have been changed and the signature no longer matches.

**AMSI is doing the real work here.** Since PowerShell 4, script content is handed to the Antimalware Scan Interface at runtime, after any encoding or wrapping has been unwound in memory, and Defender's `Cobacis` family matches the RC4 loop structure there. That's why renaming the file or reshuffling the base64 alphabet wouldn't help an attacker. AMSI sees the deobfuscated body, not the file on disk. Confirm AMSI is enabled and unbypassed on your endpoints, because a working AMSI bypass turns this from "blocked on sight" into "runs silently".

**Script block logging is the durable telemetry.** PowerShell event ID 4104 records the deobfuscated content of what actually ran. Enable it by policy and ship it to your SIEM. It survives the obfuscation for the same reason AMSI does.

**The behavioral shape is signature independent.** Hunt on what the technique structurally has to do, none of which depends on the specific bytes:

- byte array manipulation with a 256 element state array and modulo 256 arithmetic, the fingerprint of a hand rolled cipher in a script
- base64 encode or decode calls sitting next to XOR loops
- reading a file, transforming it, and writing an encoded blob back to disk

**Don't treat execution policy as a control.** The project's own instructions tell you to run `Set-ExecutionPolicy Unrestricted`, which is fair for a demo to ask and also a perfect illustration of why execution policy isn't a security boundary. It's a guardrail against accidental double clicks, trivially sidestepped with a flag. Watching for changes to it is more useful than relying on it.

The theme is the same one from the [Morse packet transceiver post]({{ site.baseurl }}{% post_url projects-posts/redteam/2026-08-16-morse-code-packet-transceiver %}). Detections pinned to a tool's specific constants are cheap to write and cheap to evade, while detections built on what a technique structurally has to do are harder to write and much harder to get around. Here the attacker can't remove the RC4 loop or the base64 without giving up the thing the tool exists to do, and AMSI and script block logging both see straight through the wrapping.

## Repository

The project is catalogued in the [Red Team Projects](https://github.com/4D5A/Red-Team-Projects) repository along with the detection guidance above and the credits to Remko Weijnen and Doug Finke. The entry there is documentation only. There's no reason to publish another copy of a decade old script that Defender quarantines on sight, and the analysis is the part with any value.

RC4 and the reversible encodings shown here provide no meaningful confidentiality. Don't use them to protect anything that actually needs protecting.

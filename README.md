![cdlogo](https://carefuldata.com/images/cdlogo.png)

# Spell Hunter

Spell Hunter is a Python module for searching out "interesting" bytes from files.

The functionality comes from the 'hunter' feature of the Rust CLI tool named [giant-spellbook](https://github.com/jpegleg/giant-spellbook/).

The patterns that Spell Hunter is searching for are various bytes that might be useful for research, including
patterns related to software exploits, vulnerabilities, malware, as well as useful items for reverse engineering.

There is a singular function named "hunt" in this module that searches a given file for all of the "interesting" bytes,
outputing a JSON with the file name, UTC time, along with any patterns matched and the byte positions in the file of those patterns.

An example ELF file will have the "elf_magic" pattern found:

```
{
  "File": "/usr/bin/avahi-publish",
  "Report time": "2026-01-28 06:04:33.710910326 UTC",
  "Matched patterns": [
    {
      "Pattern name": "elf_magic",
      "Byte offset": [0]
    }
  ]
}

```

Here is another example JSON of a normal file that has more interesting bytes:

```
{
  "File": "/usr/bin/busybox",
  "Report time": "2026-01-28 06:05:50.842018625 UTC",
  "Matched patterns": [
    {
      "Pattern name": "pe_magic",
      "Byte offset": [805278, 805314]
    },
    {
      "Pattern name": "elf_magic",
      "Byte offset": [0]
    },
    {
      "Pattern name": "gzip_magic",
      "Byte offset": [447791]
    },
    {
      "Pattern name": "zip_magic_local",
      "Byte offset": [459675]
    },
    {
      "Pattern name": "zip_magic_central",
      "Byte offset": [459645]
    },
    {
      "Pattern name": "zip_magic_end",
      "Byte offset": [459786]
    },
    {
      "Pattern name": "bin_sh_use",
      "Byte offset": [756212, 756513, 784805]
    },
    {
      "Pattern name": "shadow_access",
      "Byte offset": [696275]
    },
    {
      "Pattern name": "passwd_access",
      "Byte offset": [696263]
    }
  ]
}
```

All matches are _just known pattern matches_, not conclusions. The tool is an aide to research, it doesn't do the research for you.

## Installation

Install via PyPi with `pip`, `pipx`, or 'uv'.

```
uv add spell_hunter
```

Alternatively, compile the wheel from source and install the wheel directly.

```
maturin build
...
uv pip install target/wheels/spell_hunter-0.1.0-cp313-cp313-manylinux_2_34_x86_64.whl

```


#### Example usage

Let's start with a simple use of loading a hard-coded file and playing with the JSON:

```
import spell_hunter
import json

def main():
    hunter = spell_hunter.hunt('/bin/ls')
    a = json.loads(hunter)
    print(a['File'], "was the file")
    print("reported at", a['Report time'])
    print("which the matches of", a['Matched patterns'])

if __name__ == "__main__":
    main()

```

Next let's look at an example of taking command line arguments and investigating each file provided:

```
import spell_hunter
import json
import sys

def main():
    for arg in sys.argv[1:]:
        print(json.loads(spell_hunter.hunt(arg)))

if __name__ == "__main__":
    main()

```

When we execute this latest example, we get output like this:

```
.venv/bin/python3.13 main.py /bin/uptime /bin/bash /bin/sh /usr/local/bin/enchant
{'File': '/bin/uptime', 'Report time': '2026-01-28 05:59:05.859766527 UTC', 'Matched patterns': [{'Pattern name': 'elf_magic', 'Byte offset': [0]}]}
{'File': '/bin/bash', 'Report time': '2026-01-28 05:59:06.643373702 UTC', 'Matched patterns': [{'Pattern name': 'elf_magic', 'Byte offset': [0]}, {'Pattern name': 'bin_sh_use', 'Byte offset': [204386, 204545]}]}
{'File': '/bin/sh', 'Report time': '2026-01-28 05:59:06.722365909 UTC', 'Matched patterns': [{'Pattern name': 'elf_magic', 'Byte offset': [0]}, {'Pattern name': 'bin_sh_use', 'Byte offset': [98562]}]}
{'File': '/usr/local/bin/enchant', 'Report time': '2026-01-28 05:59:07.038956316 UTC', 'Matched patterns': [{'Pattern name': 'elf_magic', 'Byte offset': [0, 98913]}]}

```

Exported reverse engineering data from binja for various Touhou games.  Binja doesn't have nice export facilities so I just made some simple json files.

**Note about EoSD:** Addresses for EoSD may be unreliable.  The executable I am using has MD5 3878F9321F9F9F2ED3EFEF0A6F539A5E.  It is a version of 1.02h where the DAT files have been renamed to ASCII.  I cannot recall where it came from, unfortunately.

**Note:** Currently there are *many* things that are documented better in one game than in any other, and there is currently no single game whose files I would say paint a complete picture of my current understanding of how the games work.  So browse around!  Someday I may return to TH16 (or TH17) and try to add all the stuff I labeled in other games.

The notable files for each game:

* **`type-structs-own.json`** — WIP struct defs
  + Apologies for some long field names; I don't have any other way to write comments that can be "in my face" while browsing code.
* **`funcs.json`** — Labels and comments for functions that I've sufficiently renamed.
* **`statics.json`** — Labels, comments, and types for statics that I've sufficiently renamed.
* **`labels.json`** — Labels I generated in some functions for switch cases.

And some auxilliary files:

* **`type-structs-ext.json`** — Structs generated by binja.
* **`type-aliases.json`** — Typedefs, the majority of which are generated by binja.
* **`type-enums.json`** — Enums generated by binja.

Coverage varies wildly between games depending on how much I've needed to reverse each one.  My RE work has been primarily on TH16, so that has some of the most complete data.  Here's some particular files worth looking at:

* [Structs for TH16](data/th16.v1.00a/type-structs-own.json) — As noted above, this has some of the "fullest" definition of most structs in this repo. (the files for many other games have little info beyond struct sizes)
  * [For TH17](data/th17.v1.00b/type-structs-own.json) — Has a block of fields on AnmManager that I haven't yet added to TH16
  * [For TH14](data/th14.v1.00b/type-structs-own.json) — Has the most filled out Supervisor (ZUN's `MotherInf`) type and `Window` type.
    - (these are poorly filled out in TH16 because I don't like annotating types that live in static memory since I would lose the ability to write comments)
* [Funcs for TH15](data/th15.v1.00b/funcs.json) — Tons of pointdevice-related functions are labeled.
* [Funcs for TH14](data/th14.v1.00b/funcs.json) — Lots of functions that use Direct3D and DirectInput are identified.
* [Statics for TH08](data/th08.v1.00d/statics.json) — Unlike TH10 and later, most statics in TH08 are embedded directly in static memory rather than behind pointers.  Thankfully, I was able to map a whole bunch of them after happening upon a table full of "life before main" static initializers. :D

I'll update these frequently to keep up to date with my current binja database.

## Contributing

Feel free to submit corrections as issues or PRs.  In particular I can easily accept PRs that modify **`funcs.json`** or **`statics.json`**.

At this time, I am not capable of importing a modified version of `type-structs-*.json`.  The easiest way to contribute struct fields would be to provide me with C-style definitions; you can do this in an [issue](https://github.com/exphp-share/th-re-data/issues) if need be.  (ideally, these definitions should include char array spacer fields for any unknown regions or padding)

---

## Example scripts for reading the files

A lot of disassembly tools have python interfaces, so here's the basic structure of a python script you could write to read one of these files.

**funcs.json** or **statics.json**
```python
import json

with open('path/to/funcs.json') as f:
  dic = json.load(f)

for row in dic:
  ea = int(row['addr'], 16)
  name = row['name']
  # NOTE: statics.json also has a 'type' field with a string

  print(hex(ea), name)
```

**labels.json**
```python
import json

with open('/mnt/f/asd/clone/th16re-data/data/th06.v1.02h/labels.json') as f:
    all_labels = json.load(f)

for group, labels in all_labels.items():
    for addr, suffix in labels:
        # E.g. constructs full labels like  'std__case_0__posKeyframe'
        addr = int(addr, 16)
        full_label = '{}__case_{}'.format(group, suffix)

        print(hex(addr), full_label)
    print()
```

**type-structs-*.json**
```python
import json

with open('/mnt/f/asd/clone/th16re-data/data/th06.v1.02h/type-structs-own.json') as f:
    structs = json.load(f)

for struct_name, struct in structs.items():
    # convert addresses to integers
    struct = [(int(addr, 16), member, ty) for (addr, member, ty) in struct]

    print('struct {}'.format(struct_name))
    for (offset, member_name, member_ty), (next_offset, _1, _2) in zip(struct, struct[1:]):
        member_size = next_offset - offset
        if member_ty is None:
            # these bytes are unlabeled;  you might want to e.g. put a filler here
            print('  - (filler)  - {:#x} bytes'.format(member_size))

        elif member_ty == 'struct zCOMMENT[0x0]':
            # these fields are dumb workarounds to write comments sometimes.
            # You might not care about them.
            pass

        else:
            # field with known type and label.
            # Parsing the type might be difficult but you at least know its size...
            print('  - {}  - {:#x} bytes'.format(member_name, member_size))
    print()
```

If you have written scripts for importing into a tool like ghidra or IDA, please share them in an [issue](https://github.com/exphp-share/th-re-data/issues) or on Discord and I can add them or link to them here.

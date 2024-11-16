This is a super-draft spec for a gaussian splatting container file format, as initially discussed in the [MrNerf Discord server (invite link)](https://discord.com/invite/NqwTqVYVmj).  
It's started gaining some traction outside of the MrNerf discord, so we've moved it to a proper Github repo for more accessible collaboration.  

- Draft spec: [draft-spec.md](draft-spec.md)
- Discussion topics: [SharkWipf/gaussian-splatting-container-format/discussions](https://github.com/SharkWipf/gaussian-splatting-container-format/discussions)
- Todos and issues: [SharkWipf/gaussian-splatting-container-format/issues](https://github.com/SharkWipf/gaussian-splatting-container-format/issues)
- (Temporary) roadmap: [SharkWipf/gaussian-splatting-container-format/issues/2](https://github.com/SharkWipf/gaussian-splatting-container-format/issues/2)

The actual container would consist of a zip file containing the metadata file and scene files, possibly with a custom extension (tbd).  
Compression would be determined on a file-by-file basis (some formats don't compress well, and storing uncompressed would allow direct mmapping of files).  

The goal is **not** to create a one-size-fits-all format that can represent all gaussian splatting methods through metadata.  
The goal is **only** to allow storing and unpacking of different gaussian splatting files/standards along with metadata telling one what they're dealing with (hence a container format).  

Design principles:
- Flexible enough not to stifle innovation in a rapidly developing field.
- Simple enough that implementation (using a library) would take a handful lines of code at most.
- Has to support any current and future gaussian splatting output file formats, like .ply, .spz, .png, etc, as well as the different variations therein, in a way that informs the opening program what they're dealing with.
- Has to allow multiple data files per format, allowing for more complex formats, and options like storing thumbnails or even texture/mesh data along with the file when desirable.

The main reasons for a container format:
- In a very active research field like this, there are increasingly many different non-compatible file formats showing up, and many don't have the metadata required to differentiate between them.
- It's increasingly complex for both applications and users to know what they're dealing with.
- Standardizing on a single format at this point in time is likely to stifle innovation, or spin off a lot of competing standards. A non-intrusive container format would avoid that.
- Importantly, a non-intrusive container format would not conflict with any of the existing formats whatsoever, as it operates on a different level. It contains the competing formats, it does not compete with them.
- Also importantly, the cost of supporting a container format, as well as the cost of dropping it if something better ever shows up, are minimal.

The intention at this stage is to draft up a spec, then whip up some libraries for the most popular languages, make draft PRs for the most popular projects with complete implementations, and poll for feedback.  

(This section is duplicated in both README.md and draft-spec.md to ease introduction)

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

---

Keys starting with an _underscore are optional.


```
{
  "container_version": "0.0",                                        // Version of the container format.
  "packer": "SharkWipf/FooContainer",                                // Tool used to make the container.
  "packer_version": "0.0",                                           // Tool version.
  "_creation_date": "2024-11-09T12:00:00Z",                          // Creation date, optional.
  "convention_hints": ["arxiv:2312.13299", "arxiv:2403.17888"],      // Implementations in this container.
  "_meta": {                                                         // Non-functional metadata like training parameters/results. Optional.
    "steps": 30000,                                                  // Free-form.
    "method": "2dgs",                                                // Free-form.
    "psnr": 27.81,                                                   // Free-form.
  },
  "_parameters": {},                                                 // Functional, global parameters that apply to the scene as a whole and not to a single file. Optional.

  "data_entries": {                                                  // The metadata for the individual files packed in this container.
    "scene_0": {                                                     // The type and index of the data contained. A `scene_0` is always required.
      "file_name": "scene.ply.png",                                  // The file path + name relative to the root of the container.
      "created_by": "nerfstudio-project/gsplat",                     // The application that created/trained the file.
      "_created_by_version": "short commit hash or version here",    // The version of the application that created the file.
      "_parameters": {                                               // Functional, data-specific parameters that apply to just this file entry.
        "sh_layers": 3,                                              // Free-form.
        "unit": "meters",                                            // Free-form.
        "scale": "1.0"                                               // Free-form.
      }
    },
    "thumbnail_0": {                                                 // The type and index of the data contained. `thumbnail_0` and other entries are optional, only `scene_0` is required.
      "file_name": "thumbnail.png",                                  // The file path + name relative to the root of the container.
      "created_by": "nerfstudio-project/gsplat",                     // The application that created/trained the file.
      "_created_by_version": "short commit hash or version here"     // The version of the application that created the file.
    }
  }
}
```

---

### Global Metadata Fields

---

#### `container_version`

- **Type**: `string`
- **Required**: **Yes**
- **Example**: `"0.0"`
- **Description**: Specifies the version of the container format.

---

#### `packer`

- **Type**: `string`
- **Required**: **Yes**
- **Example**: `"SharkWipf/FooContainer"`
- **Description**: The name of the application or tool used to pack/create the container.

**Why?**: Helps keep track of any potential bugs down the line.

---

#### `packer_version`

- **Type**: `string`
- **Required**: **Yes**
- **Example**: `"0.0"`
- **Description**: The version of the packer application used to create the container.

---

#### `_creation_date`

- **Type**: `string` (ISO 8601 format)
- **Required**: **Optional**
- **Example**: `"2024-11-09T12:00:00Z"`
- **Description**: The date and time when the container was created.

---

#### `convention_hints`

- **Type**: `array` of `string`
- **Required**: **Yes**
- **Example**: `["arxiv:2312.13299", "arxiv:2403.17888"]`
- **Description**: References to conventions, standards, or papers telling us how to open the data.

**Why?**: Arxiv IDs are probably not the best standard for this, other things can be used as well as long as it's unique to the specific (accessing) implementation of the method used.

---

#### `_meta`

- **Type**: `object`
- **Required**: **Optional**
- **Example**:
  ```json
  {
    "steps": 30000,
    "method": "2dgs",
    "psnr": 27.81
  }
  ```
- **Description**: Contains non-functional metadata related to the training process or other attributes not directly required for loading or rendering.

**Why?**: Allows storing training settings and results along with the file, which allows for easier organizing/searching/filtering of files.

---

#### `_parameters`

- **Type**: `object`
- **Required**: **Optional**
- **Example**: `{ }`
- **Description**: Functional, global parameters that apply to the entire scene rather than individual files.

**Why?**: Not sure. I cannot actually think of any functional parameters this would currently be the place for. But the aim is to remain flexible and accomodating to future standards.

---

### Data Entries

---

#### `data_entries`

- **Type**: `object`
- **Required**: **Yes**
- **Description**: Contains metadata for each file included in the container. The keys are strings in the format `<data_type>_<index>`, where `<data_type>` represents the type of data (e.g., `scene`, `thumbnail`, `texture`, `mesh`, `model`, `data`) and `<index>` is a numeric index starting from `0`. The exact allowed values for `<data_type>` are to be determined. Each container must always have a `scene_0` data entry. Other entries will depend on the specific implementation of the format contained.

**Why?**: Unlike the previous approach, this merges 4 different entries into one, simplifying both spec and implementation. Exact allowed keys tbd, current keys are placeholders.

---

#### Data Entry Fields

Each object within `data_entries` represents a file in the container and may include the following fields:

---

##### `file_name`

- **Type**: `string`
- **Required**: **Yes**
- **Example**: `"scene1.ply.png"`
- **Description**: The file path and name relative to the root of the container.

---

##### `created_by`

- **Type**: `string`
- **Required**: **Yes**
- **Example**: `"nerfstudio-project/gsplat"`
- **Description**: The application or tool that created this data entry.

**Why?**: In a field of many parallel implementations, knowing the exact tool that made something can make all the difference. Looking for a better name though, as this is ambiguous and can be taken to mean the author.

---

##### `_created_by_version`

- **Type**: `string`
- **Required**: **Optional**
- **Example**: `"short commit hash or version here"`
- **Description**: The version of the application that created the data entry.

---

##### `_parameters`

- **Type**: `object`
- **Required**: **Optional**
- **Example**:
  ```json
  {
    "sh_layers": 3,
    "unit": "meters",
    "scale": "1.0"
  }
  ```
- **Description**: Contains functional metadata used for properly loading or interpreting the data specific to this file entry.

**Why?**: Allows for the inclusion of specific parameters that are required for viewing the data.


---

**Changelog:**
- Update 1: Remove "deflated" parameter, as it can be retrieved from the zip metadata. Add description at the top.
- Update 2: Make created_by_version optional. Add changelog.
- Update 3: Remove `_coordinate_conversion`. No real way to consistently implement this, and should be part of the specific standard's handling anyway, not of the container.
- Update 4: Shower thought: We can get rid of `data_type`, `primary_data_key` and `thumbnail_data_key` and the iffy `data_entries` array all at once by making the `data_type`s the array keys.

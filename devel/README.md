I am planning to work on this problem, and a few other features:

- array unrolling to be made optional,
- linking between table element and its description section,
- support for Markdown, AsciiDoc, reStructuredText, by choosing a Jinja template,

## Array unrolling

### Inspiration

I tried to find some data sheets containing a register map to be used as inspiration. First I checked the CSR definition for RISC-V, but those are very specific, even containing features not present in the SystemRDL standard. The other two are my first tries and they look good. I was also thinking to look into some NPX ARM Microcontrollers.

1. [Zynq UltraScale+ Devices Register Reference](https://docs.amd.com/r/en-US/ug1087-zynq-ultrascale-registers/Overview)
2. [RP2350 Datasheet](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf) chapter _2.2. Address Map_

In the Xilinx document I noticed the following:

1. The following hierarchy is used:

   _x_ Top `regmap` containing a table of `regfile` instances with links to sections _x.y_ for each `regfile` component definition/description.
   _x.y_ For each `regfile` a table of `reg` instances with links to sections _x.y.z_ for each `reg` component definition/description.
   _x.y.z_ For each `reg` a table of `field` instances, without links, the description is part of the table.

2. Arrays of `regmap` and `reg` instances are **unrolled** inside tables.
   There might be cases, where an array range is used [0:N-1], but I did not see any yet.
   The arrays were of length around 4, some of length 16.

3. There are two types of unrolling:

   1. In case the `regmap` and `reg` instances share the same definition/description section
      like [ZDMA module](https://docs.amd.com/r/en-US/ug1087-zynq-ultrascale-registers/ZDMA-Module)
      are appended the `n` suffix (no separator, index) in tables.

      In this case, there is also a list of _base addresse_ for all `regmap` instances,
      and a list of _absolute address_ for the same `reg` in all `regmap` instances.

      https://docs.amd.com/r/en-US/ug1087-zynq-ultrascale-registers/ZDMA_ERR_CTRL-ZDMA-Register

   2. Some `regmap` arrays use the `_n` suffix (`_` separator and index) and also link to separate definitions.
      For example `CORESIGHT_A53_CTI_0`/`1`/`2`/`3`.

      It seems the use of the suffix is not strict, since some registers have
      the suffix without the separator and separate definitions.

      https://docs.amd.com/r/en-US/ug1087-zynq-ultrascale-registers/CTIINEN0-A53_CTI_0-Register

   I did not check yet, whether the repeated definitions for apparently identical `regmap`/`reg` instances
   also have identical descriptions, but it appears they do.
   The _base addresse_ for `regmap` and _absolute address_ for `reg` are distinct as expected.

   There are also cases with indexes which are not arrays.
   Like control registers which do not fit into 32-bits and are therefore split into CTRL0, CTRL1.

   https://docs.amd.com/r/en-US/ug1087-zynq-ultrascale-registers/ZDMA_CH_CTRL0-ZDMA-Register

### Optional unrolling

The two different cases of array unrolling in the Xilinx document seem to be related to two approaches:

- actual instance arrays, resulting in automated indexing and shared definitions,
- multiple instances of the same component with manual indexing resulting in separate definition sections.



## Linking between table element and its description section

For example a link from the identifier of a register in a table
to the section fully documenting this register (similar for fields, ...).

Initially I thought this would be difficult, since Markdown links to headers
are constructed from the header text, and this would not work well if there are
multiple headers with the same text.
Than I noticed at least GitHub is appending an additional index (`-n`)
to the `n`-th heading with the same text.
So it would be extra work to track this indes, but it is doable.

AsciiDoc provides the [ID Attribute](https://docs.asciidoctor.org/asciidoc/latest/attributes/id/)
which provides an unique path for links.

There might be something similar for reStructuredText, I did not check yet.

## Support for multiple formatted text languages

The three document formats Markdown, AsciiDoc, reStructuredText all have a similar approach.
They are based on human readable text, and are often parsed/rendered
by version control web interfaces (GitHub, GitLab, ...).

Since all three have similar features, it would probably be possible to support
all three formats by the same Python jude, just by switching the Jinja template.

It wold probably make sense to find a new name for such a combined tool?

- PeakRDL-TextDoc,
- PeakRDL-DocText,
- PeakRDL-DocGen.




# Licensing

- [Licensing](#licensing)
  - [MIT](#mit)
    - [Redistribution](#redistribution)
    - [MIT -> Closed source: Yes](#mit---closed-source-yes)
    - [MIT -> GPL: Yes](#mit---gpl-yes)
  - [BSD](#bsd)
    - [Redistribution](#redistribution-1)
    - [BSD -> GPL: Yes](#bsd---gpl-yes)
  - [GPL](#gpl)
    - [GPL -> MIT: No](#gpl---mit-no)
  - [Apache](#apache)
    - [Apache -> MIT: Yes](#apache---mit-yes)

## MIT

General info: https://www.tawesoft.co.uk/kb/article/mit-license-faq

### Redistribution

Keep the original copyright/license in a separate file (e.g. `TETRA-LICENSE.md`), or add the copyright to the (single) license file.

In case of partial redistribution, can add:

```
Classes under the XYZ package were taken from project ABC, Copyright 1999 Original Author. These classes are licensed under the MIT license. See XYZ-LICENSE.md for additional details.
```

Reference: https://opensource.stackexchange.com/a/6981.

### MIT -> Closed source: Yes

### MIT -> GPL: Yes

MIT license requires to add some messages to the binaries:

> [...]
>
> The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
>
> [...]

Reference: https://opensource.stackexchange.com/a/8878.

## BSD

### Redistribution

Keep the original copyright/license in a separate file, or add the copyright to the (single) license file; example of the latter:

```
Copyright: 2021 Saverio miroddi <saverio.pub2@gmail.com>
Copyright assets and original (Python) code: 2019 Eben Upton <eben@raspberrypi.org>
```

The second line has been modified from the original, in order to define the scope (assets and original code).

### BSD -> GPL: Yes

See [MIT -> GPL](#mit---gpl-yes).

## GPL

### GPL -> MIT: No

## Apache

### Apache -> MIT: Yes

See [MIT -> GPL](#mit---gpl-yes).

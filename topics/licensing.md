# Licensing

- [Licensing](#licensing)
  - [MIT redistribution (also partial)](#mit-redistribution-also-partial)
  - [BSD redistribution](#bsd-redistribution)
  - [Statically linking BSD/MIT/Apache to GPL](#statically-linking-bsdmitapache-to-gpl)

## MIT redistribution (also partial)

Keep the original copyright/license in a separate file (e.g. `TETRA-LICENSE.md`), or add the copyright to the (single) license file.

In case of partial redistribution, can add:

```
Classes under the XYZ package were taken from project ABC, Copyright 1999 Original Author. These classes are licensed under the MIT license. See XYZ-LICENSE.md for additional details.
```

Reference: https://opensource.stackexchange.com/a/6981.

## BSD redistribution

Keep the original copyright/license in a separate file, or add the copyright to the (single) license file; example of the latter:

```
Copyright: 2021 Saverio miroddi <saverio.pub2@gmail.com>
Copyright assets and original (Python) code: 2019 Eben Upton <eben@raspberrypi.org>
```

The second line has been modified from the original, in order to define the scope (assets and original code).

## Statically linking BSD/MIT/Apache to GPL

Reference: https://opensource.stackexchange.com/a/8878.

MIT license requires to add some messages to the binaries:

> [...]
>
> The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
>
> [...]

GPL v2 is not compatible with Apache v2.

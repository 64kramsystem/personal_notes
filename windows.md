# Windows

- [Windows](#windows)
  - [License operations](#license-operations)

## License operations

Transfer key

```sh
slmgr /dlv # display (partial) license info

slmgr /upk  # uninstall key
slmgr /cpky # remove key from registry

slmgr /ipk  # install key; better done via GUI, which handles more cases
```

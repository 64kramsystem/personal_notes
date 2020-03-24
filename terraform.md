## Table of contents

- [State operations](#state-operations)
  - [Move resources from one statefile to another](#move-resources-from-one-statefile-to-another)

## State operations

```sh
# List resources
#
terraform state list
```

### Move resources from one statefile to another

```sh
resources=(res1 res2)

cd "$project_dest"
terraform state pull > remote.tfstate

cd "$project_src"
for resource in "${resources[@]}"; do
  terraform state mv -state-out="$project_dest/remote.tfstate" "$resource" "$resource"
done

cd "$project_dest"
terraform state push remote.tfstate
```

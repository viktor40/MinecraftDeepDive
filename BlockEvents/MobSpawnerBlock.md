# General Info
Type = 1
data = 0
Creates a block event when updating spawns


# Details
If the type is 1 and it’s client side, spawn delay = min spawn delay and true is returned. Else false is returned.

# Code
`net.minecraft.block.entity.BellBlockEntity`
```Java
@Override
public boolean onSyncedBlockEvent(int type, int data) {
    if (this.logic.setSpawnDelay(type)) {
        return true;
    }
    return super.onSyncedBlockEvent(type, data);
}
```

`net.minecraft.block.entity.BlockEntity`
```Java
// just a complicated way to return false?
public boolean onSyncedBlockEvent(int type, int data) {
        return false;
    }
```

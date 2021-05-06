# General Info
Type 1 = 1  
data = 0  
The block event is used for the teleport cooldown.

# Details
If the type is one, it will set the teleportCooldown = 40 and return true. Else it will return false.

# Code
`net.minecraft.block.entity.EndGatewayBlockEntity`
```Java
    @Override
    public boolean onSyncedBlockEvent(int type, int data) {
        if (type == 1) {
            this.teleportCooldown = 40;
            return true;
        }
        return super.onSyncedBlockEvent(type, data);
    }
```

`net.minecraft.block.entity.BlockEntity`
```Java
public boolean onSyncedBlockEvent(int type, int data) {
    return false;
}
```

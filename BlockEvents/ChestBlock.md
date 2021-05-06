# General Info
Type 1 = 1  
data = the amount of players viewing the inventory  
When opening or closing the inventory  

# Details
If the type is one, it will set viewerCount = data and return true. Else it will return false.

# Code
`net.minecraft.block.entity.ChestBlockEntity`
```java
@Override
public boolean onSyncedBlockEvent(int type, int data) {
    if (type == 1) {
        this.viewerCount = data;
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

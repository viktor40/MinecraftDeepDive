# General Info
Type = 1  
data = the amount of players viewing the inventory. 
When the enderchest gets ticked (once every second). When opening or closing the ender chest


# Details
If the type is one, it will set viewerCount = data and return true. Else it will return false

# Code
`net.minecraft.block.entity.EnderChestBlockEntity`
```Java
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
// just a complicated way to return false?
public boolean onSyncedBlockEvent(int type, int data) {
        return false;
    }
```

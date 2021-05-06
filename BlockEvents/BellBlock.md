# General Info
Type = 1
data = `direction.getId()`, where direction encomasses the side on which the bell block was hit.
A new block event is scheduled when a bell block is hit


# Details
The bell block will ignore the block event if the type isn’t 1 (return false). If the type is 1 the following happens:
1) First of all the bell memories get notified. This mostly does some variable allocations
2) Secondly, the amount of ticks the bell is resonating is set to 0.
3) The side on which the bell got hit is recorded.
4) The ringing time gets set to 0 and ringing is set to true.
5) then true gets returned.

# Code
`net.minecraft.block.entity.BellBlockEntity`
```Java
@Override
public boolean onSyncedBlockEvent(int type, int data) {
    if (type == 1) {
        this.notifyMemoriesOfBell();
        this.resonatingTicks = 0;
        this.lastSideHit = Direction.byId(data);
        this.ringTicks = 0;
        this.ringing = true;
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

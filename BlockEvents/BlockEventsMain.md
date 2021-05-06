# Order of processing:
In the block event phase, the game calls `processSyncedBlockEvents()` from `net.minecraft.server.ServerWorld`. All block events are stored in the `synchedBlockEventQueue`. This queue is an `ObjectLinkedOpenHashSet`. This is a type specific linked hash set which uses a hash table to represent a set. For more information about has sets see [Earthcomputers Video](https://www.youtube.com/watch?v=y5Cx07OHaOI). This is a linked hash set which means a list is stored as well as a hash set. This list is used for iteration.
```Java
public int hashCode() {
   int i = this.pos.hashCode();
   i = 31 * i + this.block.hashCode();
   i = 31 * i + this.type;
   i = 31 * i + this.data;
   return i;
}
```
`this.pos.hashCode():`
```Java
public int hashCode() {
   return (this.getY() + this.getZ() * 31) * 31 + this.getX();
}
```
`this.block.hashCode():`
```
public int hashCode() {
   int i = this.self.hashCode();
   i = 31 * i + this.other.hashCode();
   i = 31 * i + this.facing.hashCode();
   return i;
 }
 ```

`this.self.hashCode()`, `this.other.hashCode()` and `this.facing.hashCode()` 
Use the standard `hashCode()` method from the Object class in java. `self` and `other` are `net.minecraft.block.BlockState` objects. `facing` is a `net.minecraft.util.math.Direction` object.

This means that a block events hashCode depends on:
It’s position (x, y and z)
the type of block. All different attributes of the block are taken into account. But things like it’s position are not going to affect this particular part of the hash code. This means that this part should be the same for all blocks with the same attributes and thus should be the same for 2 exact copies of a block in different positions.
The type of block event (see next section)
the data of the block event (see next section for how data is determined)

`processSynchedBlockEvents` will go through the `syncedBlockEventQueue` until it’s empty. First there will be a check. Here the game will get the `BlockState` of the block in the position where the `BlockEvent` is called. If this `BlockState` corresponds to the block state of the `BlockEvent` then processing will continue. If not the code will go to the next object in the queue. This can happen when for example the block is broken or has changed state already.

After this the `onSynchedBlockEvents` method will be called. After this there are 2 possibilities. It’s either a block entity or a normal block. There is a difference between those two in what happens. More about this will be discussed later

# Types, data and specifics (addSynchedBlockEvent):
The order in which block events happen is both locational and directional since it is stored in a hashset.
Things that call: net.minecraft.server.world.serverworld.addSyncedBlockEvent

- Bell Block  
    Type = 1  
    data = direction.getId(), where direction encomasses the side on which the bell block was hit.  
    A new block event is scheduled when a bell block is hit.  

- Note Block  
    Type = 0  
    data = 0  
    A block event is scheduled when a note is played  

- Piston Block  
    * Extending  
        Type = 0  
        data = the id of the direction the piston is facing  

    * Retracting  
        Type = 1  
        data = the id of the direction the piston is facing  

    * Moving  
        Type = 2  
        data = the id of the direction the piston is facing  

        A piston block creates block events when it starts extending or retracting and during it’s movement.
    
- Chest Block  
Type 1 = 1  
    data = the amount of players viewing the inventory
        When opening or closing the inventory

- End Gateway Block  
    Type 1 = 1  
    data = 0  
    When it starts the teleport cooldown after teleporting an entity

- Ender Chest Block  
    Type = 1  
    data = the amount of players viewing the inventory. When the enderchest gets ticked (once every second). When opening or closing the ender chest

- Mob Spawner Block Entity  
    Type = 1  
    data = 0  
    When Updating spawns  

- Shulker Box Block Entity 
    Type = 1  
    data = the amount of players viewing the inventory  
    When opening or closing the shulker box  

# What does a block event do? (onSynchedBlockEvent)
So now we know when a block event gets scheduled, how it’s hash works, what actions schedule block events etc. But we don’t yet know what it does, why block events are used etc.

First the onSynchedBlockEvents method from AbstractBlock gets called. This will then call a different method depending on if the block is a normal block or a block entity.

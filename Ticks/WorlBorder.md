# What Happens
`net.minecraft.world.border.WorldBorder.tick` is called in the game loop.
The only thing that will happen during this “ticking” of the world border is checking if the world border needs to change in size, and then executing that.

# Code  
`ServerWorld.tick()`:
```Java
public void tick(BooleanSupplier shouldKeepTicking) {
        .
        .
        .

        profiler.push("world border");
        this.getWorldBorder().tick();
  
        .
        .
        .
}
```

`WorldBorder.tick()`
```Java
public void tick() {
        this.area = this.area.getAreaInstance();
}

@Override
public Area getAreaInstance() {
    if (this.getTargetRemainingTime() <= 0L) {
        return new StaticArea(this.newSize);
    }
    return this;
}
```

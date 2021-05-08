!! IN PROGRESS !!

# Code

`net.minecraft.server.world.ServerChunkManager`
```Java
public void tick(BooleanSupplier shouldKeepTicking) {
    this.world.getProfiler().push("purge");
    this.ticketManager.purge();
    this.tick();
    this.world.getProfiler().swap("chunks");
    this.tickChunks();
    this.world.getProfiler().swap("unload");
    this.threadedAnvilChunkStorage.tick(shouldKeepTicking);
    this.world.getProfiler().pop();
    this.initChunkCaches();
}
```

```Java
private boolean tick() {
    boolean bl = this.ticketManager.tick(this.threadedAnvilChunkStorage);
    boolean bl2 = this.threadedAnvilChunkStorage.updateHolderMap();
    if (bl || bl2) {
        this.initChunkCaches();
        return true;
    }
    return false;
}
```


```Java
private void tickChunks() {
    long worldTime = this.world.getTime();
    long timeSinceLastMobSpawn = worldTime - this.lastMobSpawningTime;
    this.lastMobSpawningTime = worldTime;
    WorldProperties levelProperties = this.world.getLevelProperties();
    boolean debugWorld = this.world.isDebugWorld();
    boolean doMobSpawning = this.world.getGameRules().getBoolean(GameRules.DO_MOB_SPAWNING);
    if (!debugWorld) {

        SpawnHelper.Info spawnEntry;
        this.world.getProfiler().push("pollingChunks");

        // Get the randomtick speed set by the gamerule
        int randomTickSpeed = this.world.getGameRules().getInt(GameRules.RANDOM_TICK_SPEED);

        // Check if the time is divisible by 400 (every 20 seconds)
        // used for passive mob spawning
        boolean timeModulus = levelProperties.getTime() % 400L == 0L;
        this.world.getProfiler().push("naturalSpawnCount");

        // Update ticket types and load levels
        int j = this.ticketManager.getSpawningChunkCount();
        this.spawnEntry = spawnEntry = SpawnHelper.setupSpawn(j, this.world.iterateEntities(), (arg_0, arg_1) -> this.ifChunkLoaded(arg_0, arg_1));
        this.world.getProfiler().pop();
        ArrayList list = Lists.newArrayList(this.threadedAnvilChunkStorage.entryIterator());
        Collections.shuffle(list);
        list.forEach(chunk -> {
            Optional optional = chunk.getTickingFuture().getNow(ChunkHolder.UNLOADED_WORLD_CHUNK).left();
            if (!optional.isPresent()) {
                return;
            }
            this.world.getProfiler().push("broadcast");
            chunk.flushUpdates((WorldChunk)optional.get());
            this.world.getProfiler().pop();
            Optional optional2 = chunk.getEntityTickingFuture().getNow(ChunkHolder.UNLOADED_WORLD_CHUNK).left();
            if (!optional2.isPresent()) {
                return;
            }


            WorldChunk worldChunk = (WorldChunk)optional2.get();
            ChunkPos chunkPos = chunk.getPos();

            // if there is a non spectator player within a 128 radius of this chunk
            if (this.threadedAnvilChunkStorage.isTooFarFromPlayersToSpawnMobs(chunkPos)) {
                return;
            }

            // Change the inhabited time (i.e. local difficulty)
            worldChunk.setInhabitedTime(worldChunk.getInhabitedTime() + timeSinceLastMobSpawn);

            // If mob spawning is enabled and if either animals or monsters can spawn and if the position is within
            // the world border
            if (doMobSpawning && (this.spawnMonsters || this.spawnAnimals) && this.world.getWorldBorder().contains(worldChunk.getPos())) {
                SpawnHelper.spawn(this.world, worldChunk, chunkPos, this.spawnAnimals, this.spawnMonsters, timeModulus);
            }
            this.world.tickChunk(worldChunk, randomTickSpeed);
        });
        this.world.getProfiler().push("customSpawners");
        if (doMobSpawning) {
            this.world.tickSpawners(this.spawnMonsters, this.spawnAnimals);
        }
        this.world.getProfiler().pop();
        this.world.getProfiler().pop();
    }
    this.threadedAnvilChunkStorage.tickPlayerMovement();
}
```

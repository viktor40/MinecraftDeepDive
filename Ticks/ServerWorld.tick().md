This method is called in `ServerWorld.tickWorlds()`
1) World border
2) Weather
    - rain
    - thunder
    - player sleeping
3) chunk stuff
    - purge unloaded chunks
    - does TACS stuff, i'm not gonna look i'm sorry
    - Ticks chunks:
        * Update chunk load levels  
        * Natural mob spawning  
        * Mobspawning by ticking spawners  
        * Random ticks  
            - Thunder: spawn skeleton horses and do lightning bolts
            - Ice and snow: do snowing and ice forming -> Randomticks blocks in the chunk
        * Player movement
    - Unload Chunks (ticks TACS)
        * write POI  
        * Unload the chunks  
        * Initialise the chunk cache  
4) block ticks
    - Cleaning out the tick cache to differentiate between old ticks and ticks that still need to happen
    - Schedule a block tick in case it's valid
5) fluid ticks
    - Cleaning out the tick cache to differentiate between old ticks and ticks that still need to happen
    - Schedule a fluid tick in case it's valid
6) raids
    - Check if ongoing
    - Do raid stuff
7) block events
    - Call the synchedBlockEvent hashmap and execute the block events stored in it
8) entities
    - If there is an ongoing dragon fight: tick the dragon
    - check despawn
    - tick entities
    - remove entities from the world
    - actually populate the spawns in the world / loads them into the world, actually
      different to the spawning in the chunk ticks
9) block entities

Code (+ some comments of mine I used to keep track of what happens):
```Java
public void tick(BooleanSupplier shouldKeepTicking) {
        boolean bl4;
        Profiler profiler = this.getProfiler();
        this.inBlockTick = true;
        profiler.push("world border");
        this.getWorldBorder().tick();

        profiler.swap("weather");
        /* This isRaining function is decided by the weather gradient:
        return (double)this.getRainGradient(1.0f) > 0.2; */
        boolean raining = this.isRaining();
        // If the dimension has skylight, do the weather cycle (only the overworld has skylight)
        if (this.getDimension().hasSkyLight()) {
            if (this.getGameRules().getBoolean(GameRules.DO_WEATHER_CYCLE)) {
                // All weather type times are random numbers to decide for how long the weather will stay the same
                int clearWeatherTime = this.worldProperties.getClearWeatherTime();
                int thunderTime = this.worldProperties.getThunderTime();
                int rainTime = this.worldProperties.getRainTime();
                boolean thundering = this.properties.isThundering();
                boolean propertiesRaining = this.properties.isRaining();

                // except from commands clearWeatherTime isn't set anywhere
                if (clearWeatherTime > 0) {
                    --clearWeatherTime;

                    // if thundering / raining: thunderTime / rainTime = 0
                    // if not thundering / raining: thunderTime / rainTime = 1
                    // -> as long as the weather is clear, thunderTime and rainTime = 1
                    thunderTime = thundering ? 0 : 1;
                    rainTime = propertiesRaining ? 0 : 1;
                    thundering = false;
                    propertiesRaining = false;

                } else {
                    /* When clear weather is done, thunder and rain time is 1. It will change the boolean from false
                    to true for both thundering and raining. The next tick thunderTime and rainTime will be 0.
                    Then it will set a random number to the time it needs to keep raining / thundering. Now we will
                    subtract 1 from that time each tick until it's 0 again. Then the weather will change again. Now
                     thunderTime isn't > 0 anymore so it will set a random number to thunderTime again. But
                     this time it won't start thundering again, since thunder to true or false is only set in the
                     first code block. */
                    if (thunderTime > 0) {
                        if (--thunderTime == 0) {
                            thundering = !thundering;
                        }

                    } else {
                        // If it's thundering, thunderTime ∈ [3600, 15 600]
                        // If it's not thundering, thunderTime ∈ [12 000, 180 000]
                        thunderTime = thundering ? this.random.nextInt(12000) + 3600 : this.random.nextInt(168000) + 12000;
                    }
                    if (rainTime > 0) {
                        if (--rainTime == 0) {
                            propertiesRaining = !propertiesRaining;
                        }
                    } else {
                        // If it's raining, thunderTime ∈ [12 000, 24 000]
                        // If it's not raining, thunderTime ∈ [12 000, 180 000]
                        rainTime = propertiesRaining ? this.random.nextInt(12000) + 12000 : this.random.nextInt(168000) + 12000;
                    }
                }
                this.worldProperties.setThunderTime(thunderTime);
                this.worldProperties.setRainTime(rainTime);
                this.worldProperties.setClearWeatherTime(clearWeatherTime);
                this.worldProperties.setThundering(thundering);
                this.worldProperties.setRaining(propertiesRaining);
            }
            /*
            Gradients are used to set the ambient light level. Has nothing to do with sky or server light. Ambient light is the colour of the sky basically, turns from blue to gray.
            Gradients are in the range [0, 1]. I'll give weather change from clear to thundering as an example.
            When the weather changes, the gradient will go up by one each tick starting from 0.01 This means the sky will have fully changed colour after 100 ticks (5 seconds).
            After this it tries to add to the gradient but the clamp will prevent it. When it changes back to clear, it'll go down by 0.01, and will be fully clear after 100 ticks.
             */
            this.thunderGradientPrev = this.thunderGradient;
            this.thunderGradient = this.properties.isThundering() ? (float)((double)this.thunderGradient + 0.01) : (float)((double)this.thunderGradient - 0.01);
            this.thunderGradient = MathHelper.clamp(this.thunderGradient, 0.0f, 1.0f);
            this.rainGradientPrev = this.rainGradient;
            this.rainGradient = this.properties.isRaining() ? (float)((double)this.rainGradient + 0.01) : (float)((double)this.rainGradient - 0.01);
            this.rainGradient = MathHelper.clamp(this.rainGradient, 0.0f, 1.0f);
        }
        // Change rain gradient aka change sky colour, but only if the player is in the overworld
        if (this.rainGradientPrev != this.rainGradient) {
            this.server.getPlayerManager().sendToDimension(new GameStateChangeS2CPacket(GameStateChangeS2CPacket.RAIN_GRADIENT_CHANGED, this.rainGradient), this.getRegistryKey());
        }
        if (this.thunderGradientPrev != this.thunderGradient) {
            this.server.getPlayerManager().sendToDimension(new GameStateChangeS2CPacket(GameStateChangeS2CPacket.THUNDER_GRADIENT_CHANGED, this.thunderGradient), this.getRegistryKey());
        }

        // The gradient has just changed. world.isRaining() is called. isRaining() will be false if the rain gradient is <= .2
        // When the gradient is low enough the isRaining() function will also return false, and not only the variable in properties will be false
        // After this the rain will start / stop and the gradient will change for all players even if they're in the end or the nether
        if (raining != this.isRaining()) {
            if (raining) {
                this.server.getPlayerManager().sendToAll(new GameStateChangeS2CPacket(GameStateChangeS2CPacket.RAIN_STOPPED, 0.0f));
            } else {
                this.server.getPlayerManager().sendToAll(new GameStateChangeS2CPacket(GameStateChangeS2CPacket.RAIN_STARTED, 0.0f));
            }
            this.server.getPlayerManager().sendToAll(new GameStateChangeS2CPacket(GameStateChangeS2CPacket.RAIN_GRADIENT_CHANGED, this.rainGradient));
            this.server.getPlayerManager().sendToAll(new GameStateChangeS2CPacket(GameStateChangeS2CPacket.THUNDER_GRADIENT_CHANGED, this.thunderGradient));
        }


        /* If all players are sleeping, set time of day to morning, wake players, and reset weather
        Reset weather sets raining and thundering to false and rainTime and ThunderTime to 0
        */
        if (this.allPlayersSleeping && this.players.stream().noneMatch(player -> !player.isSpectator() && !player.isSleepingLongEnough())) {
            this.allPlayersSleeping = false;
            if (this.getGameRules().getBoolean(GameRules.DO_DAYLIGHT_CYCLE)) {
                long l = this.properties.getTimeOfDay() + 24000L;
                this.setTimeOfDay(l - l % 24000L);
            }
            this.wakeSleepingPlayers();
            if (this.getGameRules().getBoolean(GameRules.DO_WEATHER_CYCLE)) {
                /* This is resetWeather():
                this.worldProperties.setRainTime(0);
                this.worldProperties.setRaining(false);
                this.worldProperties.setThunderTime(0);
                this.worldProperties.setThundering(false); */
                this.resetWeather();
            }
        }
        // Here ambient darkness is calculated. This has to do with the colour of the sky. Not the skylight value
        // Ambient darkness is used for mob spawning for example
        // The skylight value depends on blocks blocking sky access
        this.calculateAmbientDarkness();

        // Change the time of day each tick
        this.tickTime();

        profiler.swap("chunkSource");
        this.getChunkManager().tick(shouldKeepTicking);
        profiler.swap("tickPending");
        if (!this.isDebugWorld()) {
            this.blockTickScheduler.tick();
            this.fluidTickScheduler.tick();
        }
        profiler.swap("raid");
        this.raidManager.tick();
        profiler.swap("blockEvents");
        this.processSyncedBlockEvents();
        this.inBlockTick = false;
        profiler.swap("entities");
        boolean bl2 = bl4 = !this.players.isEmpty() || !this.getForcedChunks().isEmpty();
        if (bl4) {
            this.resetIdleTimeout();
        }
        if (bl4 || this.idleTimeout++ < 300) {
            Entity lv4;
            if (this.enderDragonFight != null) {
                this.enderDragonFight.tick();
            }
            this.inEntityTick = true;
            ObjectIterator objectIterator = this.entitiesById.int2ObjectEntrySet().iterator();
            while (objectIterator.hasNext()) {
                Int2ObjectMap.Entry entry = (Int2ObjectMap.Entry)objectIterator.next();
                Entity lv2 = (Entity)entry.getValue();
                Entity lv3 = lv2.getVehicle();
                if (!this.server.shouldSpawnAnimals() && (lv2 instanceof AnimalEntity || lv2 instanceof WaterCreatureEntity)) {
                    lv2.remove();
                }
                if (!this.server.shouldSpawnNpcs() && lv2 instanceof Npc) {
                    lv2.remove();
                }
                profiler.push("checkDespawn");
                if (!lv2.removed) {
                    lv2.checkDespawn();
                }
                profiler.pop();
                if (lv3 != null) {
                    if (!lv3.removed && lv3.hasPassenger(lv2)) continue;
                    lv2.stopRiding();
                }
                profiler.push("tick");
                if (!lv2.removed && !(lv2 instanceof EnderDragonPart)) {
                    this.tickEntity(this::tickEntity, lv2);
                }
                profiler.pop();
                profiler.push("remove");
                if (lv2.removed) {
                    this.removeEntityFromChunk(lv2);
                    objectIterator.remove();
                    this.unloadEntity(lv2);
                }
                profiler.pop();
            }
            this.inEntityTick = false;
            while ((lv4 = this.entitiesToLoad.poll()) != null) {
                this.loadEntityUnchecked(lv4);
            }
            this.tickBlockEntities();
        }
        profiler.pop();
    }
```

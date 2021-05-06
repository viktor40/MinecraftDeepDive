# Weather cycle:
In the weather section of the tick function there are 6 variable allocations.
The first one is a variable you get from a function that decides if it’s raining or not from rain gradient (the sky colour, but more later on this). The other variables are clearWeatherTime, rainTime, thunderTime, isRaining and isThundering from the world properties.

The first part of the code is an if block that checks if clearWeatherTime is bigger than 0. This part is basically a hackfix to make it so that the weather will stay clear for a certain amount of time when the /weather clear [time] command is used. In this block clearWeatherTime will count down. After this thunderTime and rainTime will be set to 1 if it’s not thundering or raining respectively. If it is thundering or raining they will be set to 0. After this thundering and raining will be set to false in the properties.
What this accomplishes will be that once clearWeatherTime has reached 0, it will start raining and thundering the next tick.

If clearWeatherTime isn’t bigger than 0, thunderTime and rainTime will be checked.
If thunderTime is bigger than 0 we’ll check if thunderTime - 1 is equal to 0. If that is the case, then the value indicating if it’s thundering (thundering) will be reversed. This means if it was thundering, it will stop thundering and if it wasn’t thundering it will start thundering.
If however thunderTime isn’t bigger than 0, a new value is set for thunder time. This value is decided by: thunderTime = thundering ? this.random.nextInt(12000) + 3600 : this.random.nextInt(168000) + 12000;

In a more readable format, this will mean that:
If it is thundering, the new thunderTime  will be between 3 600 and 15 600.
If it is not thundering, the new thunderTime  will be between 12 000 and 180 000.
Which value somewhere in this range it becomes is determined by a random factor.

The same process happens for rainTime, though the new value for rainTime is determined differently: rainTime = propertiesRaining ? this.random.nextInt(12000) + 12000 : this.random.nextInt(168000) + 12000;


In a more readable format, this will mean that:
If it is thundering, the new rainTime will be between 12 000 and 24 000.
If it is not thundering, the new rainTime will be between 12 000 and 180 000.
Which value somewhere in this range it becomes is determined by a random factor.
After this the values that were changed will be reallocated to the corresponding fields.

So now for the explanation of how this actually works. The way in which this works is a little confusing clearWeatherTime isn’t used to determine how long the weather will stay clear outside of commands. Instead what happens is:
The game starts and clearWeatherTime, rainTime and thunderTime are set to 0. raining and thundering  are set to false. Now, since everything is set to 0 it will immediately go to the part where it sets the timers, using the formulas we’ve just seen. In this case rainTime and thunderTime will be used to indicate how long it will not be raining or thundering. Both timers will count down, and only when they have reached 0 it will start raining or thundering, depending on which timer ran out. It will start this weather by flipping the boolean as we’ve seen before. After this a new time will be set for rainTime or thunderTime. This time, they’ll be used to indicate how long it will be raining or thundering. By constantly flipping the time rainTime and thunderTime represent, there’s no need to use clearWeatherTime. Thus we only have a value for this if we want to force the game to keep the weather clear for a certain amount of time no matter what.

# Rain and thunder gradients:
A whole separate part of the weather are rain and thunder gradients. These gradients are used to determine the ambientLightLevel and the colour of the sky. As you know the sky darkens when it starts to rain or thunder and brightens again when the weather becomes clear. Well these gradients will take care of that.

When the weather is clear, both gradients will be 0. The value for these gradients will always be between 0 and 1 inclusive. If it gets higher than 1 it will be set back to 1. If it gets lower than 0 it will be set back to 0. Each tick the following happens. If it is raining, each tick we’ll add 0.01 to the rain gradient. If it’s not raining we’ll subtract 0.01. This way when the sky will take 100 ticks or 5 seconds to change colour completely.

Now there is a isRaining and isThundering method, boh will determine if it’s raining or not via the gradient. These functions are used to determine the weather outside of the tick loop.

Rain will only begin to fall when the gradient is bigger than 0.2 and will only stop falling when it’s smaller than 0.2 again. In this case we’re speaking visually of course. Internally the properties are already set when the gradients start changing.

# Sleeping:
Next in this part of the tick sleeping happens. First the time of day will be set to morning if a player has just finished sleeping. It will also reset the weather if the weather cycle is enabled. Resetting the weather set rainTime and thunderTime to 0 and raining and thundering to false.

# Daylight Cycle:
Curiously, the time of day will also change within the weather section. In this part the world time will be increased by one. If daylight cycle is enabled, the time of day will also change.
What about lightning strikes?
Well lightning strikes don’t happen in this part of the tick but rather in the chunk part, which is the next part we’ll cover.


Code:
```Java
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
```

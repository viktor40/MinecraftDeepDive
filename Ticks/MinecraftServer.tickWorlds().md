This method is called in `MinecraftServer.tick()`

1) Tick the command function manager
2) Sync world time with other dimensions each 20 ticks (1 second)
3) Tick the game, i.e. call `net.minecraft.server.ServerWorld.tick`
4) Tick networkIO
5) Update player latency
6) Server GUI Refresh

Code:
```Java
protected void tickWorlds(BooleanSupplier shouldKeepTicking) {
        this.profiler.push("commandFunctions");
        this.getCommandFunctionManager().tick();
        this.profiler.swap("levels");
        for (ServerWorld lv : this.getWorlds()) {
            this.profiler.push(() -> lv + " " + lv.getRegistryKey().getValue());
            if (this.ticks % 20 == 0) {
                this.profiler.push("timeSync");
                this.playerManager.sendToDimension(new WorldTimeUpdateS2CPacket(lv.getTime(), lv.getTimeOfDay(), lv.getGameRules().getBoolean(GameRules.DO_DAYLIGHT_CYCLE)), lv.getRegistryKey());
                this.profiler.pop();
            }
            this.profiler.push("tick");
            try {
                lv.tick(shouldKeepTicking);
            }
            catch (Throwable throwable) {
                CrashReport lv2 = CrashReport.create(throwable, "Exception ticking world");
                lv.addDetailsToCrashReport(lv2);
                throw new CrashException(lv2);
            }
            this.profiler.pop();
            this.profiler.pop();
        }
        this.profiler.swap("connection");
        this.getNetworkIo().tick();
        this.profiler.swap("players");
        this.playerManager.updatePlayerLatency();
        if (SharedConstants.isDevelopment) {
            TestManager.INSTANCE.tick();
        }
        this.profiler.swap("server gui refresh");
        for (int i = 0; i < this.serverGuiTickables.size(); ++i) {
            this.serverGuiTickables.get(i).run();
        }
        this.profiler.pop();
    }
```

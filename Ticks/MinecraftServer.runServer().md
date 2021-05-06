# What happens?
This is the main server loop.

1) Initialise the loop
2) If the server is running, check if the server can keep up (i.e. sub 50 mspt)
3) If not, log a warning 
4) reate a new tick monitor
5) While the game is running:  
    - Start the tick
    - Tick the server, i.e. call `net.minecraft.server.MinecraftServer.tick`
    - Wait for the next tick
    - End the tick and stop the tick monitor

If an exception occurs this loop will shut down the server and try to create and save a crash report.
This will also create the main game thread.

# Code
```Java
protected void runServer() {
        try {
            if (this.setupServer()) {
                this.timeReference = Util.getMeasuringTimeMs();
                this.metadata.setDescription(new LiteralText(this.motd));
                this.metadata.setVersion(new ServerMetadata.Version(SharedConstants.getGameVersion().getName(), SharedConstants.getGameVersion().getProtocolVersion()));
                this.setFavicon(this.metadata);

                // this creates the game loop
                while (this.running) {
                    long l = Util.getMeasuringTimeMs() - this.timeReference;
                    if (l > 2000L && this.timeReference - this.lastTimeReference >= 15000L) {
                        long m = l / 50L;
                        LOGGER.warn("Can't keep up! Is the server overloaded? Running {}ms or {} ticks behind", (Object)l, (Object)m);
                        this.timeReference += m * 50L;
                        this.lastTimeReference = this.timeReference;
                    }
                    this.timeReference += 50L;
                    TickDurationMonitor lv = TickDurationMonitor.create("Server");
                    this.startMonitor(lv);
                    this.profiler.startTick();
                    this.profiler.push("tick");
                    this.tick(this::shouldKeepTicking);
                    this.profiler.swap("nextTickWait");
                    this.waitingForNextTick = true;
                    this.field_19248 = Math.max(Util.getMeasuringTimeMs() + 50L, this.timeReference);
                    this.method_16208();
                    this.profiler.pop();
                    this.profiler.endTick();
                    this.endMonitor(lv);
                    this.loading = true;
                }
            } else {
                this.setCrashReport(null);
            }
        }
        catch (Throwable throwable2) {
            CrashReport lv3;
            LOGGER.error("Encountered an unexpected exception", throwable2);
            if (throwable2 instanceof CrashException) {
                CrashReport lv2 = this.populateCrashReport(((CrashException)throwable2).getReport());
            } else {
                lv3 = this.populateCrashReport(new CrashReport("Exception in server tick loop", throwable2));
            }
            File file = new File(new File(this.getRunDirectory(), "crash-reports"), "crash-" + new SimpleDateFormat("yyyy-MM-dd_HH.mm.ss").format(new Date()) + "-server.txt");
            if (lv3.writeToFile(file)) {
                LOGGER.error("This crash report has been saved to: {}", (Object)file.getAbsolutePath());
            } else {
                LOGGER.error("We were unable to save this crash report to disk.");
            }
            this.setCrashReport(lv3);
        }
        finally {
            try {
                this.stopped = true;
                this.shutdown();
            }
            catch (Throwable throwable) {
                LOGGER.error("Exception stopping the server", throwable);
            }
            finally {
                this.exit();
            }
        }
    }
```

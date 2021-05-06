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

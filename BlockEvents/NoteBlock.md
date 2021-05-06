# General Info
Type = 0
data = 0
A block event is scheduled when a note is played


# Details
1) play the note
2) display the particles and make the sound in game
3) return true

# Code
```Java
    @Override
    public boolean onSyncedBlockEvent(BlockState state, World world, BlockPos pos, int type, int data) {
        int k = state.get(NOTE);
        float f = (float)Math.pow(2.0, (double)(k - 12) / 12.0);
        world.playSound(null, pos, state.get(INSTRUMENT).getSound(), SoundCategory.RECORDS, 3.0f, f);
        world.addParticle(ParticleTypes.NOTE, (double)pos.getX() + 0.5, (double)pos.getY() + 1.2, (double)pos.getZ() + 0.5, (double)k / 24.0, 0.0, 0.0);
        return true;
    }
```

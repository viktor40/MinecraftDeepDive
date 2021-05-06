# General Info
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


# Details
TBD

# Code
```Java
@Override
public boolean onSyncedBlockEvent(BlockState state, World world, BlockPos pos, int type, int data) {
    Direction directionFacing = state.get(FACING);

    /*
    Type 0 = Extending
    Type 1 = Retracting
    Type 2 = Moving
    */

    // if not client
    if (!world.isClient) {
        boolean shouldExtend = this.shouldExtend(world, pos, directionFacing);

        // retracting or moving
        if (shouldExtend && (type == 1 || type == 2)) {
            world.setBlockState(pos, (BlockState)state.with(EXTENDED, true), 2);
            return false;
        }

        // extending
        if (!shouldExtend && type == 0) {
            return false;
        }
    }

    // extending
    if (type == 0) {
        if (!this.move(world, pos, directionFacing, true)) return false;
        world.setBlockState(pos, (BlockState)state.with(EXTENDED, true), 67);
        world.playSound(null, pos, SoundEvents.BLOCK_PISTON_EXTEND, SoundCategory.BLOCKS, 0.5f, world.random.nextFloat() * 0.25f + 0.6f);
        return true;
    } else {

        // if invalid type return true
        if (type != 1 && type != 2) return true;
        BlockEntity blockEntity = world.getBlockEntity(pos.offset(directionFacing));
        if (blockEntity instanceof PistonBlockEntity) {
            ((PistonBlockEntity)blockEntity).finish();
        }

        // blockstate of moving piston can be sticky or not
        BlockState blockStateMovingPiston = (BlockState)((BlockState)Blocks.MOVING_PISTON.getDefaultState().with(
                PistonExtensionBlock.FACING, directionFacing)).with(PistonExtensionBlock.TYPE, this.sticky ? PistonType.STICKY : PistonType.DEFAULT);

        world.setBlockState(pos, blockStateMovingPiston, 20);
        world.setBlockEntity(pos, PistonExtensionBlock.createBlockEntityPiston((BlockState)this.getDefaultState().with(FACING, Direction.byId(data & 7)), directionFacing, false, true));
        world.updateNeighbors(pos, blockStateMovingPiston.getBlock());
        blockStateMovingPiston.updateNeighbors(world, pos, 2);

        // if sticky piston
        if (this.sticky) {
            PistonBlockEntity lv7;
            BlockEntity lv6;
            BlockPos lv4 = pos.add(directionFacing.getOffsetX() * 2, directionFacing.getOffsetY() * 2, directionFacing.getOffsetZ() * 2);
            BlockState lv5 = world.getBlockState(lv4);
            boolean bl2 = false;
            if (lv5.isOf(Blocks.MOVING_PISTON) && (lv6 = world.getBlockEntity(lv4)) instanceof PistonBlockEntity && (lv7 = (PistonBlockEntity)lv6).getFacing() == directionFacing && lv7.isExtending()) {
                lv7.finish();
                bl2 = true;
            }
            if (!bl2) {
                if (type == 1 && !lv5.isAir() && PistonBlock.isMovable(lv5, world, lv4, directionFacing.getOpposite(),
                        false, directionFacing) && (lv5.getPistonBehavior() == PistonBehavior.NORMAL || lv5.isOf(Blocks.PISTON) || lv5.isOf(Blocks.STICKY_PISTON))) {
                    this.move(world, pos, directionFacing, false);
                } else {
                    world.removeBlock(pos.offset(directionFacing), false);
                }
            }
        } else {
            world.removeBlock(pos.offset(directionFacing), false);
        }
        world.playSound(null, pos, SoundEvents.BLOCK_PISTON_CONTRACT, SoundCategory.BLOCKS, 0.5f, world.random.nextFloat() * 0.15f + 0.6f);
    }
    return true;
}
```

Mods to ElectrumX to support blocksonly mode where estimatesmartfee is disabled. This is needed due to mempool spam created
by ordinals craze

Check out daemon.py where I stub out the method to return -1. This will keep the electrum client happy


async def estimatefee(self, block_count, estimate_mode=None):
        await asyncio.sleep(0.001)
        return -1



Also, fixed DockerFile to get it to build correctly

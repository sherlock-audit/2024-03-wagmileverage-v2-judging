Tangy Pickle Koala

medium

# Protocol makes use of `slot0` in the LightQuoter



## Proof of Concept

Take a look at https://github.com/sherlock-audit/2024-03-wagmileverage-v2/blob/169e587af8e9dcff9dd351443e1f64933e762e1d/wagmi-leverage/contracts/LightQuoterV3.sol#L207-L225

```solidity
    function _prepareSwapCache(
        bool zeroForOne,
        address swapPool,
        SwapCache memory cache
    ) private view {
        (uint160 sqrtPriceX96, int24 tick, , , , uint8 feeProtocol, ) = IUniswapV3Pool(swapPool)
            .slot0();
        cache.zeroForOne = zeroForOne;
        cache.liquidityStart = IUniswapV3Pool(swapPool).liquidity();
        cache.feeProtocol = zeroForOne ? (feeProtocol % 16) : (feeProtocol >> 4);
        cache.fee = IUniswapV3Pool(swapPool).fee();
        cache.tickSpacing = IUniswapV3Pool(swapPool).tickSpacing();
        cache.tick = tick;
        cache.sqrtPriceX96 = sqrtPriceX96;
        cache.sqrtPriceX96Limit = zeroForOne
            ? TickMath.MIN_SQRT_RATIO + 1
            : TickMath.MAX_SQRT_RATIO - 1;
        cache.swapPool = swapPool;
    }
```

It's evident that while preparing the cache to be used for swapping, the protocol queries `sqrtPriceX96` from uniswap's `slot0`, would be key to note that the `_prepareSwapCache` function is called in all attemots to swap via `quoteExactOutputSingle` `quoteExactInputSingle` and even `calculateExactZapIn`, problem is that the price returned from slot0 is very easy to manipulate as such causeing the protocol's attempt if swapping with this sqrtprice to be extremely faulty and easily manipulatable 

## Impact

Pool lp value can be manipulated and cause other users to receive way less tokens while swapping.

## Recommended Mitigation Steps# Title

Consider using a uniswap TWAP instead of slot0.

 

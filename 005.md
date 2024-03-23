Generous Menthol Tortoise

high

# Any user can do a AAVE flashloan on behalf of `FlashLoanAggregator`

## Summary
Any user can do a AAVE flashloan on behalf of `FlashLoanAggregator` 

## Vulnerability Detail
If we look at the AAVE flashloan logic, we'll see that any user can trigger a flashloan on behalf of another wallet. 

```solidity
  function executeFlashLoanSimple(
    DataTypes.ReserveData storage reserve,
    DataTypes.FlashloanSimpleParams memory params
  ) external {
    // The usual action flow (cache -> updateState -> validation -> changeState -> updateRates)
    // is altered to (validation -> user payload -> cache -> updateState -> changeState -> updateRates) for flashloans.
    // This is done to protect against reentrance and rate manipulation within the user specified payload.

    ValidationLogic.validateFlashloanSimple(reserve);

    IFlashLoanSimpleReceiver receiver = IFlashLoanSimpleReceiver(params.receiverAddress);  // @audit - receiver is manual input
    uint256 totalPremium = params.amount.percentMul(params.flashLoanPremiumTotal);
    IAToken(reserve.aTokenAddress).transferUnderlyingTo(params.receiverAddress, params.amount);

    require(
      receiver.executeOperation(
        params.asset,
        params.amount,
        totalPremium,
        msg.sender,
        params.params
      ),
      Errors.INVALID_FLASHLOAN_EXECUTOR_RETURN
    );
```

This would allow us to enter `_excuteCallback`  with any arbitrary data we'd like. 

The data is then forwarded to the recipient we've set, by calling `wagmiLeverageFlashCallback`. 
```solidity
            // Invoke the WagmiLeverage callback function with updated parameters
            IWagmiLeverageFlashCallback(decodedDataExt.recipient).wagmiLeverageFlashCallback(
                flashBalance,
                interest,
                decodedDataExt.originData
            );
```

While this may not be currently a problem within `LiquidityBorrowingManager`, the contract is expected to be able to work with multiple `WagmiLeverage`-compatible contracts (which can be manually added). Allowing for any arbitrary data to be passed to these contracts in such ways is highly dangerous as it can lead to all sorts of unexpected behavior.

## Impact
Passing of arbitrary data to `WagmiLeverage` contracts

## Code Snippet
https://github.com/sherlock-audit/2024-03-wagmileverage-v2/blob/main/wagmi-leverage/contracts/FlashLoanAggregator.sol#L290

## Tool used

Manual Review

## Recommendation
verify the initiator address on AAVE flashloans 

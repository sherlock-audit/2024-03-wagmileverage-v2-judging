Generous Menthol Tortoise

medium

# Liquidation bonus scales exponentially instead of linearly.

## Summary
Liquidation bonus scales exponentially instead of linearly. 

## Vulnerability Detail
Let's look at the code of `getLiquidationBonus` 
```solidity
    function getLiquidationBonus(
        address token,
        uint256 borrowedAmount,
        uint256 times
    ) public view returns (uint256 liquidationBonus) {
        // Retrieve liquidation bonus for the given token
        Liquidation memory liq = liquidationBonusForToken[token];
        unchecked {
            if (liq.bonusBP == 0) {
                // If there is no specific bonus for the token
                // Use default bonus
                liq.minBonusAmount = Constants.MINIMUM_AMOUNT;
                liq.bonusBP = dafaultLiquidationBonusBP;
            }
            liquidationBonus = (borrowedAmount * liq.bonusBP) / Constants.BP;

            if (liquidationBonus < liq.minBonusAmount) {
                liquidationBonus = liq.minBonusAmount;
            }
            liquidationBonus *= (times > 0 ? times : 1);
        }
    }
```
As we can see, the liquidation bonus is based on the entire `borrowAmount`  and multiplied by the number of new loans added.
The problem is that it is unfair when the user makes a borrow against multiple lenders.

If a user takes a borrow for X against 1 lender, they'll have to pay a liquidation bonus of Y.
However, if they take a borrow for 3X against 3 lenders, they'll have to pay 9Y, meaning that taking a borrow against N lenders leads to overpaying liquidation bonus by N times. 

Furthermore, if the user simply does it in multiple transactions, they can avoid these extra fees (as they can simply call `borrow` for X 3 times and pay 3Y in Liquidation bonuses)

## Impact
Loss of funds

## Code Snippet
https://github.com/sherlock-audit/2024-03-wagmileverage-v2/blob/main/wagmi-leverage/contracts/LiquidityBorrowingManager.sol#L280

## Tool used

Manual Review

## Recommendation
make liquidation bonus simply a % of totalBorrowed

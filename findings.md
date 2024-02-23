# Loss of accuracy leading to compromised gameplay mechanics

Loss of accuracy in points calculation leads to behavior inconsistent with documentation that compromises an aspect of the gameplay mechanics allowing players to potentially gain an unfair advantage.

## Lines of code

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L528
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L445


# Vulnerability details

## Impact
Staking factor is incorrectly calculated. Causes points to be caculated incorrectly. Users can game this to optimize their stake size to give themselves an unfair advantage against users who expect the protocol to run as documented. Increasing stake should always increase points but this is not the case.

## Proof of Concept
```
uint256 stakingFactor_ = FixedPointMathLib.sqrt(
    (amountStaked[tokenId] + stakeAtRisk) / 10**18
);
```

```
      if (stakeAtRisk == 0) {
          points = stakingFactor[tokenId] * eloFactor;
      }
```

Loss of accuracy causes a stake of 15 tokens to yield the same number of points as a stake of 10 tokens. The actual value for stake factor will be floor($\sqrt{stake}$) where $stake$ is the number of tokens staked with the decimal part truncated.

for stake of:

10.0   15.0   19.0   25.0   29.0   35.0

the staking factors are

3   3   4   5   5   5

so for example there is no difference in points for a character staking 25 and 35 tokens. If the number of tokens is large this becomes less impactful.

## Recommended Mitigation Steps

it is easy to calculate properly:

```
uint256 stakingFactor_ = FixedPointMathLib.sqrt(
    (amountStaked[tokenId] + stakeAtRisk)
);
```
line 444-446 RankedBattle.sol
```
      if (stakeAtRisk == 0) {
          points = (stakingFactor[tokenId] * eloFactor )/ 10**9;
      }
```


## Assessed type

Math

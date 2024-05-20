---
title: Bonding Curve Simulator
toc: true
---

```js
import { formatEther, parseEther } from "npm:viem@2.10.8";
```

# Bonding Curve Simulator

Welcome to DegenBuddy's Bonding Curve Simulator! This tool helps you understand
how different parameters affect the bonding curve of a token.

## Bonding Curve Parameters

Select parameters below.

### Maximum Total Supply

The maximum number of tokens that will ever be in circulation.

Choose the maximum total supply of the token:

- 1 Billion
- 1 Trillion
- 500 Hundred Trillion (Shiba Inu)

```js
const maxTotalSupply = view(
  Inputs.radio(
    [
      1_000_000_000, 4_269_000_000, 6_942_000_000, 10_000_000_000,
      1_000_000_000_000, 500_000_000_000_000,
    ],
    {
      value: 4_269_000_000,
      label: "Max Total Supply",
    }
  )
);
```

### Bonding Curve Supply

Enter the percentage of the total supply that will be sold via the bonding curve:

```js
// const bondingCurvePercentage = view(
//   Inputs.range([1, 100], {
//     step: 0.01,
//     value: 23.5,
//     label: "% Total Supply",
//   })
// );

const bondingCurveSupply = view(
  Inputs.range([1_000_000_000, 2_000_000_000], {
    step: 10_000,
    value: 1_000_000_000,
    label: "Supply Out",
  })
);
```

Total supply inside bonding curve:

```js
// const bondingCurveSupply = (bondingCurvePercentage / 100) * maxTotalSupply;
view(bondingCurveSupply.toLocaleString() + " TOKEN");
```

## Linear Bonding Curve Visualization

```js
const ethPrice = view(
  Inputs.range([1000, 5000], {
    value: 2900,
    step: 1,
    label: "ETH Price",
  })
);
```

Parameters:

```js
const bondingCurveDelta = view(
  Inputs.radio([1n, 5n, 10n, 25n, 42n, 50n, 100n], {
    value: 5n,
    label: "Delta (Wei)",
  })
);
```

Visualization:

```js

```

```js
const delta = Number(formatEther(bondingCurveDelta));
const maxBuyAmountPerStep = bondingCurveSupply / 100;

// Initials
let totalBuy = 0;
let totalAmountIn = 0;
let spotPrice = 0;
const curves = [];

while (totalBuy <= bondingCurveSupply) {
  const amountOut = Math.min(
    bondingCurveSupply - totalBuy,
    maxBuyAmountPerStep
  );

  const buySpotPrice = spotPrice + delta;
  const amountIn =
    amountOut * buySpotPrice + (amountOut * (amountOut - 1) * delta) / 2;

  totalAmountIn += amountIn;
  spotPrice = spotPrice + delta * amountOut;

  curves.push({
    priceETH: spotPrice,
    priceUSD: spotPrice * ethPrice,
    supply: totalBuy / 1_000_000_000,
    totalAmountIn: totalAmountIn,
  });

  // Increase totalBuy
  if (totalBuy == bondingCurveSupply) {
    break;
  }
  totalBuy += amountOut;
}

const plotPricePerTokenUSD = Plot.plot({
  marginLeft: 100,
  y: {
    label: "Price Per Token (USD)",
    grid: true,
  },
  x: {
    label: "Supply (M)",
  },
  marks: [Plot.lineY(curves, { x: "supply", y: "priceUSD" })],
});

const plotPricePerTokenETH = Plot.plot({
  marginLeft: 100,
  y: {
    label: "Price Per Token (ETH)",
    grid: true,
  },
  x: {
    label: "Supply (B)",
  },
  marks: [Plot.lineY(curves, { x: "supply", y: "priceETH" })],
});

const plotETHRaised = Plot.plot({
  marginLeft: 100,
  y: {
    label: "ETH Inside Pool (ETH)",
    grid: true,
  },
  x: {
    label: "Supply Out (B)",
  },
  marks: [Plot.lineY(curves, { x: "supply", y: "priceETH" })],
});
```

Latest Bonding Curve State:

```js
const latestCurve = curves[curves.length - 1];
view("Price In ETH:" + latestCurve.priceETH.toFixed(18));
view("Price In wei:" + parseEther(latestCurve.priceETH.toFixed(18)).toString());
view("Price In USD:" + latestCurve.priceUSD.toFixed(18) + " USD");
view(latestCurve.totalAmountIn + " ETH");
```

Circulating Supply x Price:

```js
const mcapETH = latestCurve.priceETH * bondingCurveSupply;
const mcapUSD = latestCurve.priceUSD * bondingCurveSupply;
view(mcapETH.toLocaleString() + " ETH");
view("$" + mcapUSD.toLocaleString());
```

Fully-diluted market cap (FDMC) = price x max supply.

```js
const marketCapETH = latestCurve.priceETH * maxTotalSupply;
const marketCapUSD = latestCurve.priceUSD * maxTotalSupply;
view(marketCapETH.toLocaleString() + " ETH");
view("$" + marketCapUSD.toLocaleString());
```

Shiba inu have `$14,512,398,801` for reference.

```js
view(plotPricePerTokenUSD);
view(plotPricePerTokenETH);
```

## Dev

1 wei is equal to:

```js
const weiPrice = Number(formatEther(1n)) * ethPrice;
view(`${weiPrice.toFixed(18)} USD`);
```

Token creation fee:

```js
const tokenCreationFeeETH = "0.001";
const tokenCreationFeeWei = parseEther(tokenCreationFeeETH);
const tokenCreationFeeUSD = Number(tokenCreationFeeETH) * ethPrice;

view(
  `${tokenCreationFeeWei} Wei (${tokenCreationFeeETH} ETH) (${tokenCreationFeeUSD} USD)`
);
```

Trading fee:

```js
const tradingFeePercent = 5 / 100;

view(`${tradingFeePercent * 100}%`);
```

```js
function getAmountInETH(amountOut, buySpotPrice) {
  return amountOut * buySpotPrice + (amountOut * (amountOut - 1) * delta) / 2;
}

function getAmountOutInETH(amountIn, sellSpotPrice) {
  return amountIn * sellSpotPrice - (amountIn * (amountIn - 1) * delta) / 2;
}
```

```js
// Global states
let spotPriceInETH = 0;
let totalReserveInETH = 0;

// Alice buy the token
const aliceBuy1AmountOut = 100_000_000;
const aliceBuy1BuySpotPrice = spotPriceInETH + delta;
const aliceBuy1AmountInCurveETH = getAmountInETH(
  aliceBuy1AmountOut,
  aliceBuy1BuySpotPrice
);
const aliceBuy1FeeInEth = aliceBuy1AmountInCurveETH * tradingFeePercent;
const aliceBuy1AmountInETH = aliceBuy1AmountInCurveETH + aliceBuy1FeeInEth;

// new states
spotPriceInETH = spotPriceInETH + delta * aliceBuy1AmountOut;
totalReserveInETH += aliceBuy1AmountInCurveETH;

// Snapshot
const aliceBuy1NewSpotPriceInETH = spotPriceInETH;
const aliceBuy1NewSpotPriceInUSD = aliceBuy1NewSpotPriceInETH * ethPrice;
const aliceBuy1TotalReserveInETH = totalReserveInETH;

// Bob buy TOKEN
const bobBuy1AmountOut = 100_000_000;
const bobBuy1SpotPrice = spotPriceInETH + delta;
const bobBuy1AmountInCurveETH = getAmountInETH(
  bobBuy1AmountOut,
  bobBuy1SpotPrice
);
const bobBuy1FeeInETH = bobBuy1AmountInCurveETH * tradingFeePercent;
const bobBuy1AmountInETH = bobBuy1AmountInCurveETH + bobBuy1FeeInETH;

// Update states after buy
spotPriceInETH = spotPriceInETH + delta * bobBuy1AmountOut;
totalReserveInETH += bobBuy1AmountInCurveETH;

// Snapshot
const bobBuy1NewSpotPriceInETH = spotPriceInETH;
const bobBuy1NewSpotPriceInUSD = bobBuy1NewSpotPriceInETH * ethPrice;
const bobBuy1TotalReserveInETH = totalReserveInETH;

// Alice sell a token
const aliceSell1AmountIn = 50_000_000;
const aliceSell1SpotPrice = spotPriceInETH;
const aliceSell1AmountOutCurveInETH = getAmountOutInETH(
  aliceSell1AmountIn,
  aliceSell1SpotPrice
);
const aliceSell1FeeInETH = aliceSell1AmountOutCurveInETH * tradingFeePercent;
const aliceSell1AmountOutInETH =
  aliceSell1AmountOutCurveInETH - aliceSell1FeeInETH;

spotPriceInETH = spotPriceInETH - delta * aliceSell1AmountIn;
totalReserveInETH -= aliceSell1AmountOutCurveInETH;
```

Alice buy 100.000.000 TOKEN and need the following ETH amount:

```js
view("amountInCurve (ETH)");
view(aliceBuy1AmountInCurveETH);

view("amountInCurve (USD)");
view(aliceBuy1AmountInCurveETH * ethPrice);

view("amountInCurve (wei)");
view(parseEther(aliceBuy1AmountInCurveETH.toString()));

view("fee (ETH)");
view(aliceBuy1FeeInEth);

view("fee (USD)");
view(aliceBuy1FeeInEth * ethPrice);

view("fee (wei)");
view(parseEther(aliceBuy1FeeInEth.toString()));

view("amountIn (ETH)");
view(aliceBuy1AmountInETH);

view("amountIn (USD)");
view(aliceBuy1AmountInETH * ethPrice);

view("amountIn (wei)");
view(parseEther(aliceBuy1AmountInETH.toString()));
```

New states after Alice buy the token:

```js
view("newSpotPrice (ETH)");
view(aliceBuy1NewSpotPriceInETH.toFixed(18));

view("newSpotPrice (USD)");
view(aliceBuy1NewSpotPriceInUSD.toFixed(18));

view("newSpotPrice (wei)");
view(parseEther(aliceBuy1NewSpotPriceInETH.toFixed(18)));

view("totalReserve (ETH)");
view(aliceBuy1TotalReserveInETH);

view("totalReserve (USD)");
view(aliceBuy1TotalReserveInETH * ethPrice);

view("totalReserve (wei)");
view(parseEther(aliceBuy1TotalReserveInETH.toFixed(18)));
```

Bob buy 100.000.000 TOKEN using the following amount of ETH:

```js
view("amountInCurve (ETH)");
view(bobBuy1AmountInCurveETH);

view("amountInCurve (USD)");
view(bobBuy1AmountInCurveETH * ethPrice);

view("amountInCurve (wei)");
view(parseEther(bobBuy1AmountInCurveETH.toString()));

view("fee (ETH)");
view(bobBuy1FeeInETH);

view("fee (USD)");
view(bobBuy1FeeInETH * ethPrice);

view("fee (wei)");
view(parseEther(bobBuy1FeeInETH.toString()));

view("amountIn (ETH)");
view(bobBuy1AmountInETH);

view("amountIn (USD)");
view(bobBuy1AmountInETH * ethPrice);

view("amountIn (wei)");
view(parseEther(bobBuy1AmountInETH.toString()));
```

New states after bob buy the token:

```js
view("newSpotPrice (ETH)");
view(bobBuy1NewSpotPriceInETH.toFixed(18));

view("newSpotPrice (USD)");
view(bobBuy1NewSpotPriceInUSD.toFixed(18));

view("newSpotPrice (wei)");
view(parseEther(bobBuy1NewSpotPriceInETH.toFixed(18)));

view("totalReserve (ETH)");
view(bobBuy1TotalReserveInETH);

view("totalReserve (USD)");
view(bobBuy1TotalReserveInETH * ethPrice);

view("totalReserve (wei)");
view(parseEther(bobBuy1TotalReserveInETH.toFixed(18)));
```

Price increase from alice to bob:

```js
const buy1PriceIncreaseInETH =
  bobBuy1NewSpotPriceInETH - aliceBuy1NewSpotPriceInETH;
const buy1PriceIncreaseInUSD = buy1PriceIncreaseInETH * ethPrice;
const buy1PriceIncreasePercentage =
  (buy1PriceIncreaseInETH / aliceBuy1NewSpotPriceInETH) * 100;

view("priceIncrease (ETH)");
view(buy1PriceIncreaseInETH);

view("priceIncrease (USD)");
view(buy1PriceIncreaseInUSD);

view("priceIncrease (wei)");
view(parseEther(buy1PriceIncreaseInETH.toFixed(18)));

view("priceIncrease (%)");
view(buy1PriceIncreasePercentage);
```

Alice sold 50.000.000 TOKEN, she will get the following amount of ETH:

```js
view("amountOutCurve (ETH)");
view(aliceSell1AmountOutCurveInETH);

view("amountOutCurve (USD)");
view(aliceSell1AmountOutCurveInETH * ethPrice);

view("amountOutCurve (wei)");
view(parseEther(aliceSell1AmountOutCurveInETH.toString()));

view("fee (ETH)");
view(aliceSell1FeeInETH);

view("fee (USD)");
view(aliceSell1FeeInETH * ethPrice);

view("fee (wei)");
view(parseEther(aliceSell1FeeInETH.toString()));

view("amountOut (ETH)");
view(aliceSell1AmountOutInETH);

view("amountOut (USD)");
view(aliceSell1AmountOutInETH * ethPrice);

view("amountOut (wei)");
view(parseEther(aliceSell1AmountOutInETH.toString()));
```

## Uniswap V3 Position

```js
const wethAmount = parseEther("2.4");
const tokenAmount = parseEther(bondingCurveSupply.toString());
```

### Case 1: token0=WETH, token1

Define tokens:

```js
import { Token, WETH9 } from "npm:@uniswap/sdk-core@5.0.0";

const case1_token0 = WETH9[1];
const case1_token1 = new Token(
  1,
  "0xdAC17F958D2ee523a2206206994597C13D831ec7",
  18,
  "t1",
  "token1"
);

view(case1_token0);
view(case1_token1);
```

Define fee tier:

```js
import { FeeAmount } from "npm:@uniswap/v3-sdk@3.11.2";

const case1_feeTier = FeeAmount.HIGH;

view(case1_feeTier);
```

Define initial price:

```js
import { encodeSqrtRatioX96 } from "npm:@uniswap/v3-sdk@3.11.2";

const tokenPriceInWETH = latestCurve.priceETH.toFixed(18);
const tokenPriceInWei = parseEther(tokenPriceInWETH);
view("tokenPriceInWei");
view(tokenPriceInWei);
const case1_sqrtPriceX96 = encodeSqrtRatioX96(tokenPriceInWei.toString(), 1e18);
view("sqrtPriceX96");
view(case1_sqrtPriceX96.toString());
```

Define current tick:

```js
import { TickMath } from "npm:@uniswap/v3-sdk@3.11.2";

const case1_currentTick = TickMath.getTickAtSqrtRatio(case1_sqrtPriceX96);
view(case1_currentTick);

view("min_tick");
view(TickMath.MIN_TICK);

view("max_tick");
view(TickMath.MAX_TICK);
```

Define Pool:

```js
import { Pool } from "npm:@uniswap/v3-sdk@3.11.2";

const case1_pool = new Pool(
  case1_token0,
  case1_token1,
  case1_feeTier,
  case1_sqrtPriceX96,
  0, // liquidity doesn't matter
  case1_currentTick,
  []
);
view(case1_pool);
```

Validate pool:

```js
// Returns the current mid price of the pool in terms of token0
view("token0Price");
view(case1_pool.token0Price.toFixed(18));

// Returns the current mid price of the pool in terms of token1
view("token1Price");
view(case1_pool.token1Price.toFixed(18));
```

### Case 2: token0, token1=WETH

Define tokens:

```js
import { Token, WETH9 } from "npm:@uniswap/sdk-core@5.0.0";

const case2_token0 = new Token(
  1,
  "0xdAC17F958D2ee523a2206206994597C13D831ec7",
  18,
  "t1",
  "token1"
);
const case2_token1 = WETH9[1];

view(case2_token0);
view(case2_token1);
```

Define fee tier:

```js
const case2_feeTier = FeeAmount.HIGH;

view(case2_feeTier);
```

Define initial price:

```js
const case2_sqrtPriceX96 = encodeSqrtRatioX96(1e18, tokenPriceInWei.toString());
view("sqrtPriceX96");
view(case2_sqrtPriceX96.toString());
```

Define current tick:

```js
const case2_currentTick = TickMath.getTickAtSqrtRatio(case2_sqrtPriceX96);
view(case2_currentTick);

view("min_tick");
view(TickMath.MIN_TICK);

view("max_tick");
view(TickMath.MAX_TICK);
```

Define Pool:

```js
const case2_pool = new Pool(
  case2_token0,
  case2_token1,
  case2_feeTier,
  case2_sqrtPriceX96,
  0, // liquidity doesn't matter
  case2_currentTick,
  []
);
view(case2_pool);
```

Validate pool:

```js
view("token0Price");
view(case2_pool.token0Price.toFixed(18));

view("token1Price");
view(case2_pool.token1Price.toFixed(18));
```

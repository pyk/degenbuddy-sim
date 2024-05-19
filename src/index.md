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
    [1_000_000_000, 10_000_000_000, 1_000_000_000_000, 500_000_000_000_000],
    {
      value: 1_000_000_000,
      label: "Max Total Supply",
    }
  )
);
```

### Bonding Curve Supply

Enter the percentage of the total supply that will be sold via the bonding curve:

```js
const bondingCurvePercentage = view(
  Inputs.range([1, 100], {
    step: 1,
    value: 22,
    label: "% Total Supply",
  })
);
```

Total supply inside bonding curve:

```js
const bondingCurveSupply = (bondingCurvePercentage / 100) * maxTotalSupply;
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
  Inputs.radio([1n, 100n, 1000n, 10000n, 100000n, 200000n], {
    value: 100n,
    label: "Delta (Wei)",
  })
);
```

Visualization:

```js

```

```js
const delta = Number(formatEther(bondingCurveDelta));

console.log("bondingCurvePercentage", bondingCurvePercentage);
console.log("maxTotalSupply", maxTotalSupply);

const maxBuyAmountPerStep = bondingCurveSupply / 100;

// Initials
let totalBuy = 0;
let totalAmountIn = 0;
let spotPrice = 0;
const curves = [];

console.log("totalBuy", totalBuy);
console.log("bondingCurveSupply", bondingCurveSupply);

while (totalBuy <= bondingCurveSupply) {
  console.log("======================");
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

  console.log("totalBuy", totalBuy);
  console.log("amountOut", amountOut);

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
view(latestCurve.priceETH.toFixed(18) + " ETH");
view(latestCurve.priceUSD.toFixed(18) + " USD");
view(latestCurve.totalAmountIn + " ETH");
```

Circulating Supply x Price:

```js
console.log("maxTotalSupply", maxTotalSupply);
console.log("bondingCurveSupply", bondingCurveSupply);

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

function getAmountOutETH(amountIn, sellSpotPrice) {
  return amountIn * sellSpotPrice - (amountIn * (amountIn - 1) * delta) / 2;
}
```

```js
const amountOut = 10_000_000;
let spotPriceInETH = 0;
const buySpotPrice = spotPriceInETH + delta;
const amountInCurveETH = getAmountInETH(amountOut, buySpotPrice);
const feeInEth = amountInCurveETH * tradingFeePercent;
const amountInETH = amountInCurveETH + feeInEth;
spotPriceInETH = spotPriceInETH + delta * amountOut;

// new states
const totalReserveInETH = amountInCurveETH;
const spotPriceInETHAfterFirstBuy = spotPriceInETH;

// Alice sold all tokens
// const amountIn = 10_000_000;
// const amountOutCurveInETH = getAmountOutETH(amountIn, spotPriceInETH);
// const totalPriceDecrease = amountIn * delta;
// spotPriceInETH = spotPriceInETH - totalPriceDecrease;
// const spotPriceInETHAfterFirstSell = spotPriceInETH;
```

Alice buy 10.000.000 TOKEN and need the following ETH amount:

```js
view("amountInCurve (ETH)");
view(amountInCurveETH);

view("amountInCurve (USD)");
view(amountInCurveETH * ethPrice);

view("amountInCurve (wei)");
view(parseEther(amountInCurveETH.toString()));

view("fee (ETH)");
view(feeInEth);

view("fee (USD)");
view(feeInEth * ethPrice);

view("fee (wei)");
view(parseEther(feeInEth.toString()));

view("amountIn (ETH)");
view(amountInETH);

view("amountIn (USD)");
view(amountInETH * ethPrice);

view("amountIn (wei)");
view(parseEther(amountInETH.toString()));
```

New states after buy is executed:

```js
view("newSpotPrice (ETH)");
view(spotPriceInETHAfterFirstBuy);

view("newSpotPrice (USD)");
view(spotPriceInETHAfterFirstBuy * ethPrice);

view("newSpotPrice (wei)");
view(parseEther(spotPriceInETHAfterFirstBuy.toFixed(18)));

view("totalReserve (ETH)");
view(totalReserveInETH);

view("totalReserve (USD)");
view(totalReserveInETH * ethPrice);

view("totalReserve (wei)");
view(parseEther(totalReserveInETH.toFixed(18)));
```

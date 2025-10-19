---
title: Replacing the batteries in a Cyberpower UPS
date: 2025-10-19 14:38:11
tags: UPS, Battery, Cyberpower, DIY
---

When the batteries in my aging Cyberpower CP1500PFCLCD finally gave up the ghost, I was faced with a decision: spend nearly as much as the UPS is worth on a replacement battery pack, or find a cheaper alternative.

## The Problem

The CP1500PFCLCD is an old but still functional UPS. While official and aftermarket replacement battery packs are readily available, they cost more than the current value of the unit itself. Official Cyberpower replacement packs run around $50-70, and third-party compatible sets are in a similar price range. For a UPS that can be replaced entirely for not much more, this seemed like poor economics to install new batteries with mediocre capacity.

The stock battery configuration uses two 12V 9Ah sealed lead-acid batteries connected in series to provide the 24V system voltage. A quick voltage test confirmed the 24v configuration:

![Old battery pack voltage test showing 26.29V on a Fluke multimeter](1-old-cell-voltage.jpg)

The meter shows 26.29V on the old pack - close to nominal, but the battery runtime had dropped to a measly 5 minutes under light load.

## The Solution

Rather than purchasing a wholly new UPS, I went into the local autoparts store to look for the cheapest 12v pair I could buy. The 12V 18Ah sealed lead-acid batteries I found are double the amp-hour capacity of the stock batteries. By trading in the old batteries, I avoided paying the core fee on the new ones.

![New ES17-12 batteries from O'Reilly Auto Parts displayed with old battery cores and Super Start battery box](2-orileys-new-batteries-old-cores.jpg)

The catch? The new batteries are physically larger than the stock batteries and won't fit inside the UPS case. The solution was a cheap external battery box which provides plenty of room for the larger cells.

## Installation

The installation process is straightforward if you're comfortable working with low-voltage DC wiring:

### Preparing the External Battery Box

The first step was wiring the new batteries in series to produce the required 24V. As you can see, there's plenty of extra space in the box for future expansion:

![ES17-12 batteries installed in external battery box with initial series wiring showing extra space for future expansion](3-assembling-the-cells.jpg)

The batteries are connected in series - positive terminal of the first battery to negative terminal of the second battery. The remaining positive and negative terminals become the output that connects to the UPS.

### Routing the Leads

After completing the internal wiring, I routed the output leads through the side port of the battery box and protected them with some nylon wire loom:

![Completed battery box wiring showing series connection with blue wire loom protecting the output leads](4-new-cells-wiring-complete.jpg)

### Testing and Connection

Before connecting to the UPS, I verified the voltage output from the new battery configuration:

![Voltage test of new battery configuration showing 25.48V on Fluke multimeter, confirming proper assembly](5-new-cells-voltage-test.jpg)

The meter shows 25.48V - right where it should be for a freshly assembled 24V battery pack. With voltage confirmed and polarity verified, I connected the leads to the UPS and powered it on.

## Results

![Completed installation with external battery box connected to Cyberpower CP1500PFCLCD UPS via wire loom protected leads, UPS display showing 100% charge](6-project-complete.jpg)

The UPS immediately recognized the new batteries and began charging. The results exceeded expectations:

* Battery runtime estimate jumped from approximately 5 minutes to approximately 70 minutes
* The UPS recognized and began charging the new batteries without issue
* The external battery box has additional room for another pair of batteries if I choose to expand capacity further in the future

## Tradeoffs

There is one notable drawback to using significantly larger capacity batteries:

* The UPS takes considerably longer to fully charge the 18Ah batteries compared to the original 9Ah pack. This is expected given the doubled capacity, but it's worth noting if you experience frequent power outages.

## Conclusion

For the cost of the batteries and a battery box, I've extended the life of an otherwise functional UPS and more than doubled its runtime. While this approach sacrifices the compact form factor of the original design, it provides excellent value and leaves room for future expansion. The longer charge time is a minor inconvenience compared to the extended runtime during outages.


# Brick-API: LEGO Inventory Management Service

A Python/Tornado API service that provides LEGO set inventory management with real-time data integration and comprehensive metrics collection.

## Overview

Built for LEGO enthusiasts and collectors, this API service demonstrates how to create a specialized inventory system that integrates with external data sources while maintaining high performance and comprehensive observability.

The main idea is to provide a simple UI for rebuilding LEGO sets by looking up their inventory by set number.

![Brick-API Screenshot](https://www.nickhedberg.com/images/NiU7pYDb0cfdw2jPjtDsifRcwb0=/fit-in/1200x1200/s3-us-west-2.amazonaws.com/nick-hedberg/img%2F1824%3A2858%2F090a3e20ad74ac12efa4a0289084a944f3f564ac.png)

## Implementation

I reverse-engineered LEGO's inventory data from their missing bricks lookup tool. This service essentially proxies and displays that data, with additional local storage to track pieces users have found while rebuilding sets.

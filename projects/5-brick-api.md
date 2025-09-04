
# Brick-API: LEGO Inventory Management Service

A Python/Tornado API service that provides LEGO set inventory management with real-time data integration and comprehensive metrics collection.

## Overview

Built for LEGO enthusiasts and collectors, this API service demonstrates how to create a specialized inventory system that integrates with external data sources while maintaining high performance and comprehensive observability.

The main idea is to provide a simple UI for rebuilding LEGO sets by looking up their inventory by set number.

## Implementation

I reverse-engineered LEGO's inventory data from their missing bricks lookup tool. This service essentially proxies and displays that data, with additional local storage to track pieces users have found while rebuilding sets.

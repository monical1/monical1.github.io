---
title: issue3:整理ticketDTO
tags: issues
---

## 背景介绍

tar服务依赖了seat和stock，目前直接使用了seat/stock的DTO。但本次需求里将选座 / 非选座 / 预留 / 区域票档映射表 拆分开，对应的seat/stock DTO实体也随之变化。鉴于此。应该早就应该隔离外部服务。新建自己的DTO，防污染！！！

## 现有DTO一览

`com.maoyan.show.stock.api.dto.ReserveStockDTO` - `ReserveStock`

| **reserveStockId**        | PK           |
| ------------------------- | ------------ |
| **skuId**                 | ticketId     |
| **reserveTotalStock**     | 预留总库存   |
| **reserveSaledStock**     | 预留已售库存 |
| **reserveTag**            | 预留标签     |
| **tpId**                  |              |
| **tpSalesplanId**         |              |
| **reserveAvailableStock** | 预留可售库存 |
| **poolType**              | 票池类型     |



`com.maoyan.show.seat.client.stock.dto.ReserveStockDTO` - `ReserveStock`

| **reserveStockId**        | PK           |
| ------------------------- | ------------ |
| **skuId**                 | ticketId     |
| **reserveTotalStock**     | 预留总库存   |
| **reserveSaledStock**     | 预留已售库存 |
| **reserveTag**            | 预留标签     |
| **tpId**                  |              |
| **tpSalesplanId**         |              |
| **reserveAvailableStock** | 预留可售库存 |
| **poolType**              | 票池类型     |
| **showId**                | 场次ID       |
| **areaIdRectIdMapByTag**  | 区域Mapping  |



`com.maoyan.show.seat.client.stock.dto.ShowStockDTO` - `S_Stock`

| **stockId**             | PK               |
| ----------------------- | ---------------- |
| **salesPlanId**         |                  |
| **totalStock**          | 总库存           |
| **saleStock**           | *非预留已售库存* |
| **currentStock**        | *非预留可售库存* |
| **projectTicketId**     | 票品ID           |
| **reserveTotalStock**   | *预留总库存*     |
| **reserveSaledStock**   | *预留已售库存*   |
| **globalStockId**       |                  |
| **poolType**            |                  |
| **tpId**                |                  |
| **tpSalesplanId**       |                  |
| **showId**              |                  |
| **areaIdRectIdMap**     |                  |
| **reserveStockDTOList** | 来自S_Stock      |
| **globalStockDTO**      | 来自Global_Stock |
|                         |                  |



`com.maoyan.show.seat.client.stock.dto.StockDTO` -> `S_Stock`

| **stockId**           | PK   |
| --------------------- | ---- |
| **salesPlanId**       |      |
| **totalStock**        |      |
| **saleStock**         |      |
| **currentStock**      |      |
| **projectTicketId**   |      |
| **reserveTotalStock** |      |
| **reserveSaledStock** |      |
| **globalStockId**     |      |
| **poolType**          |      |
| **tpId**              |      |
| **tpSalesplanId**     |      |
| **showId**            |      |
| **areaIdRectIdMap**   |      |
|                       |      |
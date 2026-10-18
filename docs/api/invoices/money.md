---
title: Money
parent: Invoices
grand_parent: Money and shops
nav_order: 4
---

# Money

Invoice amounts are always whole numbers, in both currencies. This matters when you pick the amount for `Create`.

## Things to know

- `Create` takes a `MoneyCurrency` (`OC` or `RM`) and a whole `int`. The game keeps invoice amounts as whole numbers, even for [RM](../../glossary.md#rm) (Real Money), so an invoice never has a fraction. See [Money](../economy/money-cash.md#money-cash). <span class="tech">(The game stores `ItemInvoiceData.amount` as `int` and rounds RM to a whole number when it creates the invoice. The game's `Money.money` type never appears in this API.)</span>

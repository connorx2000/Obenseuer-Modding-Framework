---
title: How invoices are paid and saved
parent: Invoices
grand_parent: Money and shops
nav_order: 5
---

# How invoices are paid and saved

How the game pays and saves the invoices a mod sends. Read it before you rely on a bill being paid or removed.

## Things to know

- The player pays an invoice by putting the bill into a payment machine. The game has no other way to pay one: no bank page, computer or phone pays an invoice. <span class="tech">(The only path is `InvoicePaymentMachine.Select`, which calls `InvoiceController.PayInvoice`.)</span>
- When the bill goes into the machine, the game checks the money and takes it from the player's wallet. It then runs the invoice's follow-up action and removes the invoice from the open list.
- The open invoices are [saved](../../guides/getting-started/save-data.md) with the rest of the game. The bill item in the player's mail or bag is a separate thing that carries the same data. Cancelling or paying an invoice does not remove that item from the mailbox or inventory. <span class="tech">(`InvoiceController` is a `SavableScript`, and its public `invoices` list is saved with the global save. The item is a mail item that carries the same `ItemInvoiceData` object.)</span>

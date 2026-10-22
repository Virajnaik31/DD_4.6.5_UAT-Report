# Instructions

- Following Playwright test failed.
- Explain why, be concise, respect Playwright best practices.
- Provide a snippet of code with the fix, if possible.

# Test info

- Name: orderFulFilmentChecklist\case-021.spec.ts >> Case 021 - Full Order Fulfilment Flow
- Location: tests\orderFulFilmentChecklist\case-021.spec.ts:13:5

# Error details

```
Error: locator.waitFor: Target page, context or browser has been closed
Call log:
  - waiting for locator('tr').filter({ hasText: 'U2710000056' }).first() to be visible

```

# Test source

```ts
  520 |     for (let attempt = 0; attempt < 3; attempt++) {
  521 |         const correctInvoice = generateRandomInvoiceNumber();
  522 |         await page.getByRole('textbox', { name: 'Invoice Number' }).fill(correctInvoice);
  523 |         await page.getByRole('textbox', { name: 'Weight (KG)' }).fill('1');
  524 |         await page.getByRole('textbox', { name: 'Width (CM)' }).fill('1');
  525 |         await page.getByRole('textbox', { name: 'Height (CM)' }).fill('1');
  526 |         await page.getByRole('textbox', { name: 'Length (CM)' }).fill('1');
  527 |         const packingDeliveryStatusCombo = page.getByRole('combobox', { name: /Status/i })
  528 |             .or(page.locator('.MuiSelect-select'))
  529 |             .or(page.getByLabel('', { exact: true }))
  530 |             .first();
  531 |         await packingDeliveryStatusCombo.click();
  532 |         await page.getByRole('option', { name: /Ready To Ship|Packed/i }).first().click();
  533 |         await screenshot('packing_form_filled_correct_invoice');
  534 |         await page.getByRole('button', { name: 'SUBMIT' }).click();
  535 | 
  536 |         const okBtnPackingSubmit = page.getByRole('button', { name: 'Ok' }).or(page.getByRole('button', { name: 'OK' }));
  537 |         await okBtnPackingSubmit.waitFor({ state: 'visible' });
  538 |         await screenshot('packing_ready_to_ship_submit_popup');
  539 | 
  540 |         const isDuplicate = await page.locator('text=Duplicate Invoice').isVisible().catch(() => false);
  541 |         await okBtnPackingSubmit.click();
  542 | 
  543 |         if (!isDuplicate) {
  544 |             invoiceSuccess = true;
  545 |             break;
  546 |         }
  547 |         console.log(`⚠️ Invoice ${correctInvoice} was duplicate, retrying with fresh unique invoice...`);
  548 |         await page.waitForTimeout(1000);
  549 |     }
  550 | 
  551 |     await page.getByRole('link', { name: 'Dashboard' }).click();
  552 |     await page.getByRole('link', { name: 'Orders Packed And Sent To OFT' }).first().click();
  553 |     await page.getByRole('textbox', { name: 'Search' }).fill(intentId);
  554 |     await page.getByRole('columnheader', { name: 'ACTION' }).click();
  555 |     const viewDetailsPackedFinal = page.locator('table tbody tr').filter({ hasText: intentId }).getByRole('button', { name: /View Details/i }).first();
  556 |     await viewDetailsPackedFinal.scrollIntoViewIfNeeded().catch(() => { });
  557 |     await viewDetailsPackedFinal.click();
  558 |     await page.getByRole('heading', { name: 'SHIPPING / BILLING ADDRESS' }).click();
  559 |     await screenshot('packing_final_details');
  560 |     await page.getByRole('button', { name: 'Account' }).click();
  561 |     await page.getByRole('menuitem', { name: 'Sign out' }).click();
  562 | 
  563 |     // ── OFT login (Order Packed ➔ Ready To Ship) ──────────────────────────────
  564 |     await page.getByRole('textbox', { name: 'Email Address' }).fill('oft+3025.alex.john@gmail.com');
  565 |     await page.getByRole('textbox', { name: 'Password' }).fill('oft+3025.alex.john@gmail.com');
  566 |     await page.getByRole('button', { name: 'Sign In' }).click();
  567 | 
  568 |     await page.getByRole('button', { name: 'Order Packing Status' }).click();
  569 |     await page.getByRole('link', { name: 'Order Packed', exact: true }).first().click();
  570 |     await page.getByRole('textbox', { name: 'Search' }).fill(intentId);
  571 |     await page.getByRole('columnheader', { name: 'ACTION' }).click();
  572 | 
  573 |     const viewDetailsBtn = page.locator('table tbody tr').filter({ hasText: intentId }).getByRole('button', { name: /View Details/i }).first();
  574 | 
  575 |     await viewDetailsBtn.waitFor({ state: 'visible', timeout: 15000 });
  576 |     await viewDetailsBtn.click();
  577 |     await screenshot('oft_order_packed_details');
  578 | 
  579 |     // For Send to DealsDray RM, Order Packed details directly has Remark and READY TO SHIP button
  580 |     const remarkInput = page.getByRole('textbox', { name: 'Remark' }).or(page.locator('textarea[name="remark"]')).first();
  581 |     await remarkInput.waitFor({ state: 'visible', timeout: 15000 });
  582 |     await remarkInput.fill('Confirmed');
  583 |     await screenshot('oft_process_remark_filled');
  584 | 
  585 |     await page.getByRole('button', { name: 'READY TO SHIP', exact: true }).click();
  586 |     await page.getByRole('button', { name: 'OK' }).waitFor({ state: 'visible' });
  587 |     await screenshot('oft_process_ready_to_ship_success');
  588 |     await page.getByRole('button', { name: 'OK' }).click();
  589 | 
  590 |     // ── Out Station Order Ready To Ship ───────────────────────────────────────
  591 |     const outStationReadyLink = page.getByRole('link', { name: 'Out Station order Ready To Ship', exact: true }).first();
  592 |     if (!(await outStationReadyLink.isVisible().catch(() => false))) {
  593 |         const orderReadyBtn = page.getByRole('button', { name: 'Order Ready To Ship Status', exact: true });
  594 |         await orderReadyBtn.scrollIntoViewIfNeeded().catch(() => { });
  595 |         await orderReadyBtn.click().catch(() => { });
  596 |     }
  597 |     await outStationReadyLink.scrollIntoViewIfNeeded().catch(() => { });
  598 |     await outStationReadyLink.click();
  599 |     await page.getByRole('textbox', { name: 'Search' }).fill(intentId);
  600 | 
  601 |     await page.getByRole('combobox', { name: 'Logistics' }).click();
  602 |     const logisticsOption = page.getByRole('option', { name: /SHIPROCKET/i }).or(page.getByRole('option').first()).first();
  603 |     await logisticsOption.waitFor({ state: 'visible', timeout: 15000 });
  604 |     await logisticsOption.click();
  605 | 
  606 |     // Select Courier option on manifestation page
  607 |     await page.getByRole('combobox', { name: 'Select Courier' }).click();
  608 |     await page.waitForTimeout(500);
  609 | 
  610 |     await page.pause();
  611 |     const targetCourierOpt1 = page.getByRole('option', { name: /DTDC|Blue\s*Dart|Shadowfax|Delhivery/i }).or(page.getByRole('option').first()).first();
  612 |     const courierKeyword = await targetCourierOpt1.innerText().catch(() => 'DTDC');
  613 |     console.log(`📌 Selected Courier for Manifestation: ${courierKeyword}`);
  614 |     await targetCourierOpt1.click();
  615 | 
  616 |     await page.getByRole('button', { name: 'SEARCH' }).click();
  617 |     await page.waitForTimeout(1000);
  618 | 
  619 |     const matchRow1 = page.locator('tr').filter({ hasText: intentId }).first();
> 620 |     await matchRow1.waitFor({ state: 'visible', timeout: 15000 });
      |                     ^ Error: locator.waitFor: Target page, context or browser has been closed
  621 |     const chk1 = matchRow1.getByRole('checkbox').or(matchRow1.locator('.MuiCheckbox-root, input[type="checkbox"]')).first();
  622 |     await chk1.click({ force: true }).catch(() => { });
  623 |     const chkInput1 = matchRow1.locator('input[type="checkbox"]').first();
  624 |     if (await chkInput1.count() > 0) {
  625 |         await chkInput1.check({ force: true }).catch(() => { });
  626 |     }
  627 | 
  628 |     const reqManifestBtn = page.getByRole('button', { name: 'REQUEST MANIFEST' });
  629 |     if (await reqManifestBtn.isDisabled().catch(() => false)) {
  630 |         await matchRow1.locator('td').first().click({ force: true }).catch(() => { });
  631 |         await chk1.click({ force: true }).catch(() => { });
  632 |     }
  633 | 
  634 |     await screenshot('oft_out_station_ready_to_ship_searched');
  635 |     await screenshot('oft_out_station_manifest_checked');
  636 | 
  637 |     await reqManifestBtn.click();
  638 |     await page.getByRole('button', { name: 'OK' }).waitFor({ state: 'visible' });
  639 |     await screenshot('oft_out_station_request_manifest_success');
  640 |     await page.getByRole('button', { name: 'OK' }).click();
  641 | 
  642 |     await page.getByRole('button', { name: 'Account' }).click();
  643 |     await page.getByRole('menuitem', { name: 'Sign out' }).click();
  644 | 
  645 |     // ── OFTMGR login (Release Manifest) ──────────────────────────────────────
  646 |     await page.getByRole('textbox', { name: 'Email Address' }).fill('oftmgr+3025.alex.john@gmail.com');
  647 |     await page.getByRole('textbox', { name: 'Password' }).fill('oftmgr+3025.alex.john@gmail.com');
  648 |     await page.getByRole('button', { name: 'Sign In' }).click();
  649 | 
  650 |     await page.getByRole('link', { name: 'Manifest Requests', exact: true }).first().click();
  651 |     await page.getByRole('textbox', { name: 'Search' }).fill(intentId);
  652 |     await screenshot('oftmgr_manifest_requests_list');
  653 | 
  654 |     const viewOrdersPromise = page.waitForEvent('popup', { timeout: 3000 }).catch(() => null);
  655 |     await page.getByRole('button', { name: 'View Orders' }).click();
  656 |     const popupPage = await viewOrdersPromise;
  657 |     const targetPage = popupPage || page;
  658 | 
  659 |     const releaseBtn = targetPage.getByRole('button', { name: /RELEASE MANIFEST/i }).first();
  660 |     await releaseBtn.waitFor({ state: 'visible', timeout: 15000 });
  661 |     await releaseBtn.click({ force: true });
  662 | 
  663 |     const okReleaseBtn = targetPage.getByRole('button', { name: 'OK' }).or(targetPage.getByRole('button', { name: 'Ok' })).or(targetPage.getByRole('button', { name: /OK/i })).first();
  664 |     await okReleaseBtn.waitFor({ state: 'visible', timeout: 10000 }).catch(() => { });
  665 |     if (await okReleaseBtn.isVisible().catch(() => false)) {
  666 |         await okReleaseBtn.click().catch(() => { });
  667 |         await targetPage.locator('.MuiBackdrop-root, [role="dialog"]').first().waitFor({ state: 'detached', timeout: 10000 }).catch(() => { });
  668 |     }
  669 |     await page.waitForTimeout(1500);
  670 | 
  671 |     if (popupPage && !popupPage.isClosed()) {
  672 |         await popupPage.close().catch(() => { });
  673 |     }
  674 | 
  675 |     await page.getByRole('link', { name: 'Dashboard' }).click();
  676 |     await page.getByRole('link', { name: 'Manifest Released', exact: true }).first().click();
  677 |     await page.getByRole('textbox', { name: 'Search' }).fill(intentId);
  678 |     await page.getByRole('columnheader', { name: 'ACTION' }).click();
  679 |     await screenshot('oftmgr_manifest_released_verified');
  680 | 
  681 |     await page.getByRole('button', { name: 'Account' }).click();
  682 |     await page.getByRole('menuitem', { name: 'Sign out' }).click();
  683 | 
  684 |     // ── OFT login (Print Manifest & Shipped) ──────────────────────────────────
  685 |     await page.getByRole('textbox', { name: 'Email Address' }).fill('oft+3025.alex.john@gmail.com');
  686 |     await page.getByRole('textbox', { name: 'Password' }).fill('oft+3025.alex.john@gmail.com');
  687 |     await page.getByRole('button', { name: 'Sign In' }).click();
  688 | 
  689 |     const outStationPrintLink = page.getByRole('link', { name: 'Out-Station Order Pending For Manifest Print', exact: true }).first();
  690 |     if (!(await outStationPrintLink.isVisible().catch(() => false))) {
  691 |         const orderReadyBtn = page.getByRole('button', { name: 'Order Ready To Ship Status', exact: true });
  692 |         await orderReadyBtn.scrollIntoViewIfNeeded().catch(() => { });
  693 |         await orderReadyBtn.click().catch(() => { });
  694 |     }
  695 |     await outStationPrintLink.scrollIntoViewIfNeeded().catch(() => { });
  696 |     await outStationPrintLink.click();
  697 |     await page.getByRole('textbox', { name: 'Search' }).fill(intentId);
  698 | 
  699 |     await page.getByRole('combobox', { name: 'Logistics' }).click();
  700 |     const logisticsOption2 = page.getByRole('option', { name: /SHIPROCKET/i }).or(page.getByRole('option').first()).first();
  701 |     await logisticsOption2.waitFor({ state: 'visible', timeout: 15000 });
  702 |     await logisticsOption2.click();
  703 | 
  704 |     // Select the same courier option
  705 |     await page.getByRole('combobox', { name: 'Select Courier' }).click();
  706 |     await page.waitForTimeout(500);
  707 |     const targetCourierOpt2 = page.getByRole('option', { name: courierKeyword })
  708 |         .or(page.getByRole('option').filter({ hasText: new RegExp(courierKeyword, 'i') }))
  709 |         .or(page.getByRole('option', { name: /DTDC|Blue\s*Dart|Shadowfax|Delhivery/i }))
  710 |         .or(page.getByRole('option').first())
  711 |         .first();
  712 |     await targetCourierOpt2.click();
  713 | 
  714 |     await page.getByRole('button', { name: 'SEARCH' }).click();
  715 |     await page.waitForTimeout(1000);
  716 | 
  717 |     const matchRow2 = page.locator('tr').filter({ hasText: intentId }).first();
  718 |     await matchRow2.waitFor({ state: 'visible', timeout: 15000 });
  719 |     const chk2 = matchRow2.getByRole('checkbox').or(matchRow2.locator('.MuiCheckbox-root, input[type="checkbox"]')).first();
  720 |     await chk2.click({ force: true }).catch(() => { });
```
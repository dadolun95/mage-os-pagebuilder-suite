---
layout: default
title: Store Manager Guide
parent: MageOS AdvancedWidget
nav_order: 1
---

# AdvancedWidget — Store Manager Guide

This guide explains how to use **MageOS AdvancedWidget** from the Magento Admin panel. No coding knowledge required.

---

## Overview

**MageOS AdvancedWidget** extends the Magento widget system with advanced configuration capabilities. The main feature is **multi-row settings**: widgets can expose repeatable sections where store managers can add, edit, sort, and remove an unlimited number of rows — each with its own set of fields.

Additional field types are also available: image picker, product picker, and select fields, giving agencies the tools to build rich, data-driven widget configurations without writing custom Page Builder UI components.

### For agencies

The module is designed to let development teams build advanced CMS widgets with complex, repeatable configurations and a broad set of field types. Combined with **MageOS PageBuilderWidget**, it becomes possible to create fully custom Page Builder components — complete with canvas preview and multi-row settings — without the need to develop custom Page Builder UI components from scratch.

---

## How to Use

When a widget built with AdvancedWidget is added to a CMS page or block, its configuration panel may include the following elements:

**Title separators** — visual section dividers to group related fields.

**Repeatable sections** — one or more expandable areas where you can add multiple rows of data. Each row can be:
- filled in directly from the main configuration panel or via a detail modal
- sorted by dragging the row handle
- validated on required fields before saving

**Specialized field types** available inside rows:
- **Image field** — opens the Magento media gallery to pick or upload an image
- **Product field** — opens a product search to associate a product to the row
- **Select field** — a dropdown with predefined options

---

## FAQ

**Is this module able to work together with PageBuilderWidget?**

Of course — it was developed for that scope! AdvancedWidget depends on MageOS PageBuilderWidget and the two modules are designed to be used together to build custom Page Builder components with advanced configurations.

**Are multi-row configurations working together with template import/export?**

Absolutely. No need to specify anything special to make it work — multi-row data is handled transparently by the import/export process.

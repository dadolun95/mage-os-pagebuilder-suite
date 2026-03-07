---
layout: default
title: Developer Guide
parent: MageOS AdvancedWidget
nav_order: 2
---

# AdvancedWidget — Developer Guide

Technical documentation for developers working with **MageOS AdvancedWidget**.

---

## Installation

1. Install the module into your Mage-OS / Magento 2 project with Composer:
   ```bash
   composer require mage-os/module-advanced-widget
   ```

2. Enable the module and run setup:
   ```bash
   bin/magento module:enable MageOS_AdvancedWidget
   bin/magento setup:upgrade
   ```

> **Note:** `MageOS_PageBuilderWidget` is an explicit dependency and will be installed automatically.

---

## How It Works — Features & Configuration

### 1. Title Separators

You can add visual section separators inside widget configuration panels to group related fields.

![Title section](doc/title-section_screenshot.png)

---

### 2. Repeatable Sections (Multi-row)

The main feature of this module is the ability to define **repeatable sections** — configuration areas where the store manager can add an unlimited number of rows, each containing its own set of fields.

![Repeatable section](doc/repeatable-section_screenshot.png)

#### 2.1 Tooltips

Each field inside a repeatable row can be given a dedicated tooltip to guide the user.

![Repeatable section — tooltips](doc/repeatable-section-tooltip_screenshot.png)

#### 2.2 Validation

Fields can be marked as required. The form will validate and prompt the user to fill in mandatory fields before saving.

![Repeatable section — validation](doc/repeatable-section-validation_screenshot.png)

#### 2.3 Sorting

Rows inside a repeatable section can be reordered by the store manager via a drag handle.

![Repeatable section — sorting](doc/repeatable-section-sorter_screenshot.png)

#### 2.4 Inline vs. Modal editing

For each field you can control whether it is editable directly in the main row or only inside a detail modal.

![Repeatable section — row/modal](doc/repeatable-section-row_screenshot.png)

---

### 3. Image Field

An image picker field is available for use inside repeatable rows. It opens the Magento media gallery, allowing the user to select or upload an image.

![Image field](doc/image-field_screenshot.png)

![Image field — selection](doc/image-field-selection_screenshot.png)

---

### 4. Select Field

A dropdown select field is available for use inside repeatable rows, supporting predefined option lists.

![Select field](doc/select-field_screenshot.png)

---

### 5. Product Field

A product picker field is available for use inside repeatable rows. It opens a product search grid, allowing the user to associate a product to the row.

![Product field](doc/product-field_screenshot.png)

![Product field — selection](doc/product-field-selection_screenshot.png)

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

## Implementing Multi-row Parameters

### 1. Create the Rows class

Create a PHP class in your module extending `MageOS\AdvancedWidget\Block\WidgetField\Rows` and define the `$rows` property. Each key in the array is the field name; its value is an array of field options.

```php
<?php
declare(strict_types=1);

namespace Vendor\MyModule\Block\Adminhtml\MyWidget;

use MageOS\AdvancedWidget\Block\WidgetField\Rows;

class Item extends Rows
{
    protected $rows = [
        'image' => [
            'label'       => 'Desktop Image',
            'type'        => 'image',
            'description' => 'Image used on desktop.',
            'required'    => true,
            'preview'     => true,
        ],
        'title' => [
            'label'    => 'Title',
            'type'     => 'text',
            'required' => false,
            'preview'  => true,
        ],
        'title_tag' => [
            'label'    => 'Title tag',
            'type'     => 'select',
            'options'  => ['h2' => 'H2', 'h3' => 'H3', 'p' => 'Paragraph'],
            'required' => false,
            'preview'  => false,
        ],
    ];
}
```

### 2. Register the parameter in widget.xml

Wire the class as a `block` parameter in your `widget.xml`. The parameter name **must** start with `repeatable_` for the value to be correctly encoded/decoded.

```xml
<parameter name="repeatable_my_items" xsi:type="block" sort_order="10" required="false" visible="true">
    <label translate="true">My items</label>
    <block class="Vendor\MyModule\Block\Adminhtml\MyWidget\Item"/>
</parameter>
```

### 3. Add a Title Separator (optional)

Use `MageOS\AdvancedWidget\Block\Adminhtml\Renderer\Title` to insert a visual section header. The label is passed via the `description` node as an HTML string.

```xml
<parameter name="my_section_title" xsi:type="block" required="false" visible="true" sort_order="9">
    <description><![CDATA[<h3 style="margin: 40px 0px; text-align: center;">My Section</h3>]]></description>
    <block class="MageOS\AdvancedWidget\Block\Adminhtml\Renderer\Title"/>
</parameter>
```

---

## Field Types Reference

Each entry in `$rows` supports the following keys:

| Key | Required | Description |
|---|---|---|
| `label` | yes | Field label shown in the admin UI |
| `type` | yes | Field type (see table below) |
| `required` | yes | Whether the field must be filled before saving (`true`/`false`) |
| `preview` | yes | Whether the field value is shown in the main repeatable row summary (`true`) or only in the detail modal (`false`) |
| `description` | no | Tooltip text shown next to the field label |
| `options` | only for `select` | Associative array of `value => label` pairs |

### Available types

| Type | Description |
|---|---|
| `text` | Single-line text input |
| `textarea` | Multi-line text area |
| `image` | Image picker — opens the Magento media gallery |
| `select` | Dropdown — requires the `options` key with an associative array |
| `product` | Product picker — opens a product search grid; the field name must start with `product` for the resolved SKU, name, and thumbnail to be available at render time |

### Real-world examples from Widgetkit

**Slider / Grid item** (`MageOS\Widgetkit\Block\Adminhtml\Slider\Item`, `MageOS\Widgetkit\Block\Adminhtml\Grid\Item`):
uses `image`, `text`, `textarea`, and `select` types. The `image` field with `'preview' => true` displays the thumbnail in the row summary; `title` and `content` also set `'preview' => true` to appear inline.

**Slideshow item** (`MageOS\Widgetkit\Block\Adminhtml\Slideshow\Item`):
same structure as Slider but adds a `navigation_thumb` image field and sets `'required' => true` on the main `image` field.

**Product Grid / Product Slider item** (`MageOS\Widgetkit\Block\Adminhtml\ProductGrid\Item`, `MageOS\Widgetkit\Block\Adminhtml\ProductSlider\Item`):
uses only `product` type with `'required' => true` and `'preview' => true`. The field name `product` triggers automatic resolution of `product_sku`, `product_name`, and `product_image` at render time.

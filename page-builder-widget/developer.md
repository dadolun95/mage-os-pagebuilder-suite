---
layout: default
title: Developer Guide
parent: MageOS PageBuilderWidget
nav_order: 2
---

# PageBuilderWidget — Developer Guide

Technical documentation for developers working with **MageOS PageBuilderWidget**.

---

## Installation

1. Install the module into your Mage-OS / Magento 2 project with Composer:
   ```bash
   composer require mage-os/module-page-builder-widget
   ```

2. Enable the module and run setup:
   ```bash
   bin/magento module:enable MageOS_PageBuilderWidget
   bin/magento setup:upgrade
   ```

---

## How It Works — Widget Preview

### Widget Preview assignment

To add a preview for a widget, create a `widget.xml` file in your module and change the XSD reference from the standard Magento one to the one provided by this module.

Replace:
```xml
<widgets xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Widget:etc/widget.xsd">
```
with:
```xml
<widgets xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="urn:magento:module:MageOS_PageBuilderWidget:etc/widget.xsd">
```

This unlocks the following new XML nodes inside each `<widget>` definition:

- `previewTemplates`
- `previewCss`
- `previewJs`
- `previewBlock`
- `previewBlockArguments`

### `previewTemplates`

This is the core node. It maps each frontend widget template to a corresponding admin preview template (phtml file).

```xml
<widget id="catalog_product_link">
    <previewTemplates>
        <previewTemplate name="product/widget/link/link_block.phtml" xsi:type="string">
            MageOS_PageBuilderWidget::widget-preview/product/widget/link/link_block.phtml
        </previewTemplate>
        <previewTemplate name="product/widget/link/link_inline.phtml" xsi:type="string">
            MageOS_PageBuilderWidget::widget-preview/product/widget/link/link_inline.phtml
        </previewTemplate>
    </previewTemplates>
</widget>
```

The `name` attribute of each `<previewTemplate>` must match the value of the corresponding template option declared in the original `widget.xml` `template` parameter. When that template is selected in the widget settings, the mapped preview phtml is rendered on the canvas.

### `previewCss`

Allows specifying a dedicated CSS file loaded for the widget preview. Use sufficiently specific selectors to avoid conflicts with other components.

```xml
<widget id="products_list">
    <previewTemplates>...</previewTemplates>
    <previewCss>MageOS_PageBuilderWidget::css/widget/preview/products_list_and_grid.css</previewCss>
</widget>
```

> **Important:** include the following snippet inside your preview phtml to trigger CSS (and JS) asset inclusion:
> ```php
> <?= $block->getChildHtml("previewAssets"); ?>
> ```

### `previewJs`

Allows specifying a dedicated RequireJS file for the widget preview. Mouse events are not triggered on preview elements, so this is suited for animations only (e.g. slider auto-scroll).

```xml
<widget id="products_list">
    <previewTemplates>...</previewTemplates>
    <previewJs>MageOS_PageBuilderWidget/js/my-widget-preview-js-file</previewJs>
</widget>
```

### `previewBlock`

Replaces the default Block class used to render the preview. Useful when the widget's own block class is not suitable for the admin preview context.

```xml
<widget id="products_list">
    <previewTemplates>...</previewTemplates>
    <previewBlock>MageOS\PageBuilderWidget\Block\Adminhtml\Widget\Preview\ProductsList</previewBlock>
</widget>
```

### `previewBlockArguments`

Injects additional objects into the preview block without replacing the entire block class. The injected object is accessible in the phtml via `$block->getData('argumentName')`. No need to implement `ArgumentInterface`.

```xml
<widget id="products_list">
    <previewTemplates>...</previewTemplates>
    <previewBlockArguments>
        <argument name="viewModel" xsi:type="object">
            MageOS\PageBuilderWidget\ViewModel\Adminhtml\Widget\Preview\ProductImagePreview
        </argument>
    </previewBlockArguments>
</widget>
```

Usage inside the preview phtml:

```php
/** @var ProductImagePreview $productImagePreview */
$productImagePreview = $block->getData('viewModel');
// ...
<img src="<?= $productImagePreview->getProductImage($_item) ?>" width="75" height="75" />
```

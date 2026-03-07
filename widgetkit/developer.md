---
layout: default
title: Developer Guide
parent: MageOS Widgetkit
nav_order: 2
---

# Widgetkit — Developer Guide

Technical documentation for developers working with **MageOS Widgetkit**.

---

## Installation

1. Install the module into your Mage-OS / Magento 2 project with Composer:
   ```bash
   composer require mage-os/module-widgetkit
   ```

2. Enable the module and run setup:
   ```bash
   bin/magento module:enable MageOS_Widgetkit
   bin/magento setup:upgrade
   ```

> **Dependencies:** `MageOS_PageBuilderWidget` and `MageOS_AdvancedWidget` are installed automatically.

---

## How to Develop a New Widget

The following walkthrough describes how to build a new Widgetkit-style widget from scratch. The **Slideshow** widget is used as the reference example throughout. The recommended approach is to copy an existing widget and adapt it.

---

### Step 1 — Declare the widget in `widget.xml`

Every Magento widget starts with a declaration in `etc/widget.xml`. Widgetkit uses the extended XSD provided by `MageOS_PageBuilderWidget`, which adds preview-related nodes on top of the standard Magento schema.

**Official widget.xml reference:**
- [Adobe Commerce widget.xml documentation](https://developer.adobe.com/commerce/frontend-core/guide/widgets/)
- [Mage-OS widget schema](https://github.com/mage-os/mageos-magento2/blob/main/app/code/Magento/Widget/etc/widget.xsd)

**Minimal structure:**

```xml
<widgets xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="urn:magento:module:MageOS_PageBuilderWidget:etc/widget.xsd">

    <widget id="mageos_slideshow" class="MageOS\Widgetkit\Block\Widgets\Slideshow">
        <label translate="true">MageOS Slideshow</label>
        <description translate="true">MageOS slideshow widget</description>
        <parameters>
            <!-- template selector — required for PageBuilderWidget preview mapping -->
            <parameter name="template" sort_order="1" xsi:type="select" visible="true">
                <label>Template</label>
                <options>
                    <option name="default" value="widget/hyva/slideshow.phtml" selected="true">
                        <label translate="true">MageOS Hyvä Slideshow Template</label>
                    </option>
                </options>
            </parameter>

            <!-- plain text / select parameters -->
            <parameter name="mageos_slideshow_title" sort_order="10" visible="true" xsi:type="text">
                <label>Title</label>
            </parameter>

            <!-- title separator (AdvancedWidget) -->
            <parameter name="mageos_slideshow_content_section_title"
                       xsi:type="block" required="false" visible="true" sort_order="20">
                <description><![CDATA[<h3 style="margin:40px 0;text-align:center;">Content</h3>]]></description>
                <block class="MageOS\AdvancedWidget\Block\Adminhtml\Renderer\Title"/>
            </parameter>

            <!-- repeatable rows (AdvancedWidget) -->
            <parameter name="repeatable_slideshow_items"
                       xsi:type="block" sort_order="21" required="false" visible="true">
                <label translate="true">Slideshow items</label>
                <block class="MageOS\Widgetkit\Block\Adminhtml\Slideshow\Item"/>
            </parameter>
        </parameters>

        <!-- PageBuilderWidget preview declarations -->
        <previewTemplates>
            <previewTemplate name="widget/hyva/slideshow.phtml" xsi:type="string">
                MageOS_Widgetkit::slideshow.phtml
            </previewTemplate>
        </previewTemplates>
        <previewBlock>MageOS\Widgetkit\Block\Adminhtml\Slideshow\Preview</previewBlock>
        <previewCss>MageOS_Widgetkit::css/widget/preview/hyva/slideshow.css</previewCss>
    </widget>

</widgets>
```

Key points:
- The `class` attribute on `<widget>` is the frontend Block class (`Block/Widgets/Slideshow.php`).
- The `template` parameter value (`widget/hyva/slideshow.phtml`) must match the `name` attribute of the corresponding `<previewTemplate>` node.
- `repeatable_*` parameter names are required by `MageOS_AdvancedWidget` for correct encode/decode of row data.

---

### Step 2 — Define the repeatable rows with AdvancedWidget

The item configuration for each repeatable section is a PHP class extending `MageOS\AdvancedWidget\Block\WidgetField\Rows`. Declare the `$rows` property with the fields for each row.

```php
// Block/Adminhtml/Slideshow/Item.php
namespace MageOS\Widgetkit\Block\Adminhtml\Slideshow;

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
            'label'   => 'Title',
            'type'    => 'text',
            'required' => false,
            'preview'  => true,
        ],
        'title_tag' => [
            'label'   => 'Title tag',
            'type'    => 'select',
            'options' => ['h2' => 'H2', 'h3' => 'H3', 'p' => 'Paragraph'],
            'required' => false,
            'preview'  => false,
        ],
        // ... more fields
    ];
}
```

See the [AdvancedWidget Developer Guide](../advanced-widget/developer.md) for the full field types reference.

---

### Step 3 — Implement the Preview Block

The preview block is declared via `<previewBlock>` in `widget.xml`. In Widgetkit, each preview block extends the corresponding frontend widget block and overrides `renderMainTemplate()` to **emulate the frontend area** before rendering.

```php
// Block/Adminhtml/Slideshow/Preview.php
namespace MageOS\Widgetkit\Block\Adminhtml\Slideshow;

use Magento\Store\Model\App\Emulation;
use MageOS\Widgetkit\Block\Widgets\Slideshow;

class Preview extends Slideshow
{
    public function __construct(
        protected Emulation $emulation,
        protected Conditions $conditions,
        protected Context $context
    ) {
        return parent::__construct($conditions, $context);
    }

    public function renderMainTemplate(): string
    {
        $this->emulation->startEnvironmentEmulation(
            1,
            \Magento\Framework\App\Area::AREA_FRONTEND,
            true
        );
        $mainTemplate = parent::renderMainTemplate();
        $this->emulation->stopEnvironmentEmulation();
        return $mainTemplate;
    }
}
```

**Why frontend emulation is necessary**

Widget block classes and templates often depend on frontend-area services (store configuration, theme resolution, frontend-specific helpers). When the preview is rendered inside the admin panel, Magento is running in `adminhtml` area. Calling `startEnvironmentEmulation` temporarily switches the environment to `frontend`, ensuring that template paths, image URLs, and other area-sensitive data resolve exactly as they would for a customer. Emulation is stopped immediately after rendering to avoid side effects on the rest of the admin request.

**Why templates live in `view/base`**

The base templates (e.g. `view/base/templates/widget/hyva/slideshow/templates/template.phtml`) are placed in `view/base` rather than `view/frontend` or `view/adminhtml` because Magento resolves `base` area templates as a fallback for **all** areas. This means the same template file is used both on the frontend (when a customer views the page) and in the admin preview (after environment emulation switches the area to `frontend`). There is no need to duplicate templates across areas.

---

### Step 4 — The preview phtml (`view/adminhtml/templates`)

The adminhtml preview template (e.g. `view/adminhtml/templates/slideshow.phtml`) is the entry point rendered by PageBuilderWidget on the canvas. It is placed in `view/adminhtml` because it is only ever rendered in the admin context — it is not a frontend template.

It must do three things:

1. **Call `previewAssets`** — loads the CSS declared in `<previewCss>` and the JS declared in `<previewJs>`:
   ```php
   <?= $this->getChildHtml("previewAssets"); ?>
   ```

2. **Call `addTwindManagement`** — initiates TWIND CSS generation for this widget instance:
   ```php
   <?= $this->addTwindManagement($uniqId, 'slideshow-widget'); ?>
   ```

3. **Wrap the output in a uniquely identified container** and delegate actual rendering to the Block class:
   ```php
   <div class="slideshow-widget">
       <?= $this->addTwindManagement($uniqId, 'slideshow-widget'); ?>
       <?= $this->getChildHtml("previewAssets"); ?>
       <div id="widget<?= $uniqId; ?>">
           <?= $this->renderMainTemplate(); ?>
       </div>
   </div>
   ```

The `$uniqId` (generated with `uniqid()`) ensures that when multiple widget instances of the same type are present on the canvas, TWIND scopes the compiled CSS to the correct container and does not generate conflicts between instances.

---

### Step 5 — TWIND: runtime Tailwind CSS in the admin

The Page Builder admin panel does not run the Tailwind CSS build pipeline, so utility classes in preview templates would be unstyled by default. Widgetkit solves this by bundling **TWIND** — a runtime Tailwind compiler that generates CSS from class names present in the DOM.

**How TWIND is loaded**

On `cms_page_edit` and `cms_block_edit` layout handles, Widgetkit injects Alpine.js and the two TWIND library files (`twind.min.js` and `twind/sheet.min.js`) into the page via a custom `require_js.phtml` template:

```xml
<!-- view/adminhtml/layout/cms_page_edit.xml -->
<referenceBlock name="require.js" template="MageOS_Widgetkit::page/js/require_js.phtml">
    <arguments>
        <argument name="alpinejs_path" xsi:type="string">Hyva_Theme::js/alpine3.min.js</argument>
        <argument name="twindjs_path" xsi:type="string">MageOS_Widgetkit::js/twind.min.js</argument>
        <argument name="twindjs-sheet_path" xsi:type="string">MageOS_Widgetkit::js/twind/sheet.min.js</argument>
    </arguments>
</referenceBlock>
```

**How TWIND generates scoped CSS per widget instance**

`addTwindManagement()` in `HyvaWidget` renders `view/base/templates/widget/hyva/twind.phtml`, which executes the following steps at widget load time on the canvas:

```js
// 1. create a virtual (in-memory) stylesheet
const { create } = window.twind;
const { virtualSheet } = window.TwindSheets;
const sheet = virtualSheet();
const { tw } = create({ sheet, rules: [...] });

// 2. walk every element inside the widget container (e.g. #widgetABC123)
//    and collect all Tailwind classes
const target = document.querySelector('#widget' + uid);
collectTailwindClasses(target); // recurses into child nodes
tailwindClasses.forEach(cls => tw(cls)); // compile each class

// 3. prefix every generated rule with the widget CSS class
//    (e.g. `.pagebuilder-stage-wrapper .slideshow-widget`)
//    so styles are scoped and do not leak into the rest of the admin UI
const prefix = '.pagebuilder-stage-wrapper .' + widgetCssClass;
style.textContent = sheet.target.map(rule => {
    // rewrite selectors to include the prefix
}).join('\n');
document.head.appendChild(style);
```

The generated `<style>` tag is given a stable id (`widget{uid}-style`) so it can be replaced cleanly when the widget is re-rendered (e.g. after a settings change).

This mechanism means every Tailwind class present in the rendered HTML is compiled and injected as scoped CSS — matching the styles the widget would have on the Hyvä frontend — with no build step required.

---

### Step 6 — Base templates and `resolveExpression`

The template files under `view/base/templates/widget/hyva/` contain the actual HTML markup shared between frontend and preview. The main template for the Slideshow (`slideshow/templates/template.phtml`) uses two utilities:

**Child block rendering methods**

Each `Block/Widgets` class exposes dedicated `render*` methods (`renderMainTemplate`, `renderNavTemplate`, `renderItems`, `renderItemMedia`, `renderItemContent`, etc.). Each method creates a new instance of the same block class, assigns a template and data, and returns the rendered HTML. This pattern:

- keeps individual template files small and focused on a single responsibility
- avoids passing large data structures through template variables
- makes it easy to override a single sub-template in a custom module by extending the Block class and swapping the template path in the constructor

Example from `Block/Widgets/Slideshow.php`:

```php
public function renderItems(array $itemsSettings): string
{
    return $this->getLayout()->createBlock(self::class)
        ->setTemplate($this->_itemsTemplate)
        ->setData([
            'params'   => $this->getData('params'),
            'settings' => $itemsSettings,
            'items'    => $this->getData('items'),
        ])->toHtml();
}
```

The `items` array is populated from the repeatable field by calling `getRepeatableField('repeatable_slideshow_items')` — a helper defined in `AbstractColumns` that decodes the serialised row data stored by AdvancedWidget.

**`resolveExpression`**

`resolveExpression` is defined in `MageOS\AdvancedWidget\Block\Widgets\Template` and provides a concise way to build conditional class strings and inline styles from widget parameter values without long chains of `if`/`else` in templates.

It accepts an array of expressions and the widget `$params` array. Each entry is evaluated against the params and the results are joined with spaces.

Syntax:
- A **plain string** (no `{}`) is always included.
- `{paramName}` — included only if `$params['paramName']` is truthy; replaced with the param value.
- `{@paramName: value}` — included only if `$params['paramName']` equals `value`.
- `{!paramName}` — included only if `$params['paramName']` is falsy (negation).
- `{0}` — replaced with the `$condition` value passed as the array value for that key.

Example from `template.phtml`:

```php
// $itemsSettings['class'] resolves to a space-separated list of Tailwind classes
// based on the current widget parameter values
$itemsSettings = [
    'class' => $this->resolveExpression([
        'snap-track',
        'h-auto {@mageos_slideshow_height: auto}',
        'h-screen {@mageos_slideshow_height: viewport}',
        'relative mx-auto {@mageos_slideshow_max_height}'
    ], $params),
    'style' => $this->resolveExpression([
        'min-height: {0}; {@mageos_slideshow_height: auto}' => $params['mageos_slideshow_min_height'] . 'px',
        'max-height: {0}; {@mageos_slideshow_max_height}'   => $params['mageos_slideshow_max_height'] . 'px',
    ], $params)
];
```

If `mageos_slideshow_height` is `auto`, the resolved class string will include `snap-track h-auto`. If it is `viewport`, it will include `snap-track h-screen` instead. This keeps template logic declarative and easy to follow.

---

### Step 7 — `Block/Widgets` class hierarchy

```
MageOS\AdvancedWidget\Block\Widgets\Template        (resolveExpression, stripTags)
  └── MageOS\AdvancedWidget\Block\Widgets\AbstractColumns  (getRepeatableField, getImageUrl, …)
        └── MageOS\Widgetkit\Block\Widgets\HyvaWidget      (addTwindManagement)
              ├── MageOS\Widgetkit\Block\Widgets\Slideshow  (render* methods, template paths)
              │     └── MageOS\Widgetkit\Block\Adminhtml\Slideshow\Preview  (emulation)
              ├── MageOS\Widgetkit\Block\Widgets\Slider
              ├── MageOS\Widgetkit\Block\Widgets\Grid
              └── MageOS\Widgetkit\Block\Widgets\ProductWidget  (product collection, Hyva list item)
                    ├── MageOS\Widgetkit\Block\Widgets\ProductSlider
                    └── MageOS\Widgetkit\Block\Widgets\ProductGrid
```

Each level adds a focused set of responsibilities:
- `Template` — expression evaluation utilities
- `AbstractColumns` — repeatable field decoding, image/media URL helpers
- `HyvaWidget` — TWIND management
- Concrete widget blocks — template paths and render methods
- Preview blocks — frontend area emulation

---

### Hyva Config Registration

To ensure the Tailwind CSS build on the frontend includes classes from this module's templates, the module registers itself with the Hyvä config system via an observer on `hyva_config_generate_before`:

```php
// Observer/RegisterModuleForHyvaConfig.php
$extensions[] = ['src' => substr($modulePath, strlen(BP) + 1)];
$config->setData('extensions', $extensions);
```

This tells the Hyvä Tailwind build to scan the module's source files for utility classes, preventing purge from stripping classes used only in widget templates.

---
layout: default
title: Store Manager Guide
parent: MageOS Widgetkit
nav_order: 1
---

# Widgetkit — Store Manager Guide

This guide explains how to use **MageOS Widgetkit** from the Magento Admin panel. No coding knowledge required.

---

## Overview

**MageOS Widgetkit** is a collection of ready-to-use CMS widgets built and optimised for the **Hyvä** open-source frontend theme. Each widget ships with a full Page Builder preview: when you place a widget on the canvas you see a live, styled representation of the component — including Tailwind CSS styles and Alpine.js interactions — without leaving the admin.

Previews are generated using **TWIND**, a runtime Tailwind CSS compiler. When a widget is loaded on the Page Builder stage, TWIND scans the rendered HTML for Tailwind utility classes and compiles the corresponding CSS on the fly, scoped to the widget container. Alpine.js bindings are also initialised on the preview, so animated components such as slideshows and sliders play correctly inside the canvas.

### For agencies

The module is designed to be extended. Each widget is self-contained and follows a consistent pattern, making it straightforward for a development team to copy an existing widget as a starting point and build a new, fully customised one — complete with admin configuration, repeatable rows, and a live preview. See the [Developer Guide](developer.md) for the full walkthrough.

---

## Available Widgets

| Widget | Description |
|---|---|
| **MageOS Slideshow** | Full-width image slideshow with navigation, autoplay, and configurable transitions |
| **MageOS Slider** | Content slider with images, titles, text, and call-to-action buttons |
| **MageOS Grid** | Responsive image/content grid with configurable column layout |
| **MageOS Product Slider** | Horizontal slider populated with manually selected products |
| **MageOS Product Grid** | Responsive grid populated with manually selected products |

---

## How to Use

1. Open any Page Builder-enabled content area (CMS Page, CMS Block, etc.).
2. In the Page Builder panel locate the **"CMS Widget"** component and drag it onto the canvas.
3. In the widget settings panel, select the desired Widgetkit widget from the dropdown.
4. Fill in the widget parameters — general settings, repeatable content items, layout options.
5. The canvas preview updates to show the widget with live Tailwind styles and Alpine.js behaviour.
6. Save your content as usual.

---

## FAQ

**Previews are shown with the default Hyvä style. Is it possible to customise them using project-specific CSS, similar to the frontend?**

Yes. Each widget preview accepts a dedicated CSS file that is loaded inside the Page Builder stage. See the [PageBuilderWidget Developer Guide — previewCss](../page-builder-widget/developer.md#previewcss) section to understand how to supply a custom stylesheet scoped to your widget's container class.

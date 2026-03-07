---
layout: default
title: Developer Guide
parent: MageOS PageBuilder Template Import/Export
nav_order: 2
---

# PageBuilder Template Import/Export — Developer Guide

Technical documentation for developers working with **MageOS PageBuilder Template Import/Export**.

---

## Installation

1. Install the module into your Mage-OS / Magento 2 project with Composer:
   ```bash
   composer require mage-os/module-page-builder-template-import-export
   ```

2. Run setup upgrade:
   ```bash
   bin/magento setup:upgrade
   ```

3. **Register the queue consumer** (recommended)

   The module uses a message queue consumer (`pbTemplateImport`) for async template import processing. Add it to your `env.php`:

   ```php
   <?php
   return [
       // ...
       'queue' => [
           'consumers_wait_for_messages' => 1,
       ],
       'cron_consumers_runner' => [
           'cron_run' => true,
           // ...
           'consumers' => [
               // ...
               'pbTemplateImport',
               // ...
           ],
       ],
       // ...
   ];
   ```

   This ensures import jobs are processed reliably in the background by the Magento cron consumer runner rather than inline during the request.

---

## Features

### Export Template

Once the module is enabled, templates can be exported from the admin UI via **Content > Elements > Templates** — use the **Actions** column on any template row.

Export is also available via CLI:

```bash
php bin/magento mage-os:pagebuilder_template:export
```

The export produces a `.zip` file containing the template data, ready to be transferred to another Magento instance.

---

### Import Template

Templates can be imported from the admin UI from the same **Content > Elements > Templates** page, using the **Import Template** button. The modal accepts a `.zip` file previously produced by the export feature.

Import is also available via CLI:

```bash
php bin/magento mage-os:pagebuilder_template:import
```

---

### Remote Template Import (Dropbox)

When one or more Dropbox apps are configured, the Import Template modal additionally lists all templates available in those remote repositories. Users can filter and import any remote template directly into the current Magento instance.

Remote repositories are kept in sync through two mechanisms:

- **Webhooks** — when a webhook endpoint is registered on the Dropbox app, the Magento instance is notified and updated in real time on every change to the Dropbox folder. Endpoint: `https://www.mysite.com/pagebuildertemplateie/template_remote/sync`
- **Scheduled cron** — a daily job at midnight performs a full alignment. This is the fallback when webhooks are not available (e.g. when the Dropbox account is owned by a third party).

A full manual re-sync can also be triggered at any time via CLI:

```bash
php bin/magento mage-os:pagebuilder_template:update-remote-list
```

See the [Store Manager Guide — Configuration](store-manager.md#configuration) for the full Dropbox setup instructions.

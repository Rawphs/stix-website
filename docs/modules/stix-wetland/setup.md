---
title: stix-wetland
sidebar_label: Setup
id: wetland-setup
---

## Setup

If you initialized a new stix project using the [boards cli stix preset](https://github.com/SpoonX/boards-preset-stix), stix-wetland will already be included in your project and you can move on to the [using section](#using). If not, keep reading.

1. In your stix project, simply run `yarn add stix-wetland`.
2. Add the module to your project's `src/config/modules.ts`:

```ts
import { ModuleManagerConfigInterface } from 'stix';
import { Wetland } from 'stix-wetland';

export const modules: ModuleManagerConfigInterface = [
  Wetland,
  // Your other modules.
];
```

### Setting up wetland

Configuring wetland can be done by adding a `wetland` key to your config.

1. Add `export * from './wetland';` to your `src/config/index.ts`.
2. Create a file at `src/config/wetland.ts` and add `export const wetland = {}` to it.
3. Configure wetland to your heart's desire.

You can find more info about the configuration options in the [wetland documentation](https://wetland.spoonx.org/configuration.html).

_**Note:** Due to the nature of stix's module system, all configs conveniently get merged. This means that you could add a `config/wetland.ts` to each of your modules specifying the `entityPaths[]` for those module's entities._

# Config

stix-wetland doesn't come with a whole lot of special configuration options apart from the ones wetland provides.

There's one additional configuration option, which is `devMigrations` which by default this looks at the `NODE_ENV` or `WETLAND_DEV_MIGRATIONS`.
As long as `NODE_ENV` is not `'production'` or `WETLAND_DEV_MIGRATIONS` is set, stix-wetland will run your dev migrations for you.

Read more about the configuration options in the [wetland documentation](https://wetland.spoonx.org/configuration.html).


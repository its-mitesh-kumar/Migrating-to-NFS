# Migrating Plugins to the New Frontend System in Backstage

## 🎯 Goal

Provide a complete and detailed guide for plugin authors to migrate from the legacy plugin architecture to the new [Frontend System](https://backstage.io/docs/frontend-system/) in Backstage, using a more declarative and modular approach.

---

### 📌 Overview

* Transition from imperative plugin structure to a modern, declarative structure
* Replace old plugin constructs with new extension blueprints like `PageBlueprint`, `ApiBlueprint`, and `ComponentBlueprint`
* Use `createFrontendPlugin` for plugin registration
* Allow parallel support for both legacy and new frontend systems via an `alpha.tsx` entry point
* Enable plugins to integrate seamlessly into federated apps built with the new Frontend System

---

### ✅ Migration Strategy: Step-by-Step

#### Step 1: Understand Legacy Plugin Definition

Legacy plugins used `createPlugin` from `@backstage/core-plugin-api`, manually declaring routes, APIs, and components.

```ts
import { createPlugin } from '@backstage/core-plugin-api';

export const myPlugin = createPlugin({
  id: 'my-plugin',
  apis: [ /* ... */ ],
  routes: { /* ... */ },
  externalRoutes: { /* ... */ },
});
```

Limitations:

* Tight coupling to legacy routing
* Manual route/component management
* Poor code-splitting and federation support

---

#### Step 2: Create `alpha.tsx` for New Plugin Definition

Introduce `alpha.tsx` to coexist with legacy and support the new frontend system.

```ts
import { createFrontendPlugin } from '@backstage/frontend-plugin-api';
import { convertLegacyRouteRefs } from '@backstage/core-compat-api';

export default createFrontendPlugin({
  pluginId: 'my-plugin',
  extensions: [ /* APIs, Pages, Components */ ],
  routes: convertLegacyRouteRefs({ /* legacy routeRefs */ }),
  externalRoutes: convertLegacyRouteRefs({ /* external legacy routeRefs */ }),
});
```

📌 **Explanation**: `alpha.tsx` allows maintaining compatibility with legacy while enabling migration to the new system. This helps in gradual adoption and safe rollout of the new architecture without breaking existing functionality.

---

#### Step 3: Update `package.json`

Expose `alpha.tsx` entrypoint for consumers:

```json
"exports": {
  ".": "./src/index.ts",
  "./alpha": "./src/alpha.tsx",
  "./package.json": "./package.json"
},
"typesVersions": {
  "*": {
    "alpha": [ "src/alpha.tsx" ],
    "package.json": [ "package.json" ]
  }
}
```

📌 **Explanation**: This lets plugin users explicitly import your plugin via `my-plugin/alpha` and ensures that the TypeScript type system understands the new entry point.

---

#### Step 4: Migrate Pages with `PageBlueprint`

Update pages to the declarative `PageBlueprint` API.

##### Legacy Code:

```ts
export const FooPage = fooPlugin.provide(
  createRoutableExtension({
    name: 'FooPage',
    component: () => import('./components').then(m => m.FooPage),
    mountPoint: rootRouteRef,
  }),
);
```

##### New Code:

```ts
import { PageBlueprint } from '@backstage/frontend-plugin-api';
import { compatWrapper, convertLegacyRouteRef } from '@backstage/core-compat-api';

const fooPage = PageBlueprint.make({
  params: {
    path: '/foo',
    routeRef: convertLegacyRouteRef(rootRouteRef),
  },
  loader: () =>
      import('./components/').then(m =>
        // The compatWrapper utility allows you to keep using @backstage/core-plugin-api in the
        // implementation of the component and switch to @backstage/frontend-plugin-api later.
        compatWrapper(<m.FooPage />),
      ),
});
```

✅ **Benefits**:

* Lazy loading improves performance
* Routes become strongly typed
* Pages are easier to organize and refactor
* Future-ready and optimized for federated apps

---

#### Step 5: Migrate APIs with `ApiBlueprint`

Declare APIs in a modular and reusable manner.

```ts
import { ApiBlueprint } from '@backstage/frontend-plugin-api';

const myApi = ApiBlueprint.from({
  factory: myApiFactory,
});
```

✅ **Benefits**:

* Clear separation of API concerns
* Easier testing and reuse across plugins
* Better dependency declaration and decoupling

---

#### Step 6: Migrate Components with `ComponentBlueprint`

Expose widgets and reusable UI components declaratively.

```ts
import { ComponentBlueprint } from '@backstage/frontend-plugin-api';

const MyWidget = ComponentBlueprint.make({
  async lazy() {
    // This will dynamically import the widget component.
    const { MyWidgetComponent } = await import('./widgets');
    return { Component: <MyWidgetComponent /> };
  },
});
```

✅ **Benefits**:

* Clean separation of logic
* Supports dynamic loading for performance gains
* Simplifies component sharing and embedding

---

### 🧠 Tips

* Use `convertLegacyRouteRef()` from `@backstage/core-compat-api` to bridge old routeRefs
* Wrap legacy components using `compatWrapper()` to ensure backward compatibility
* Avoid removing legacy plugin code until migration is fully complete and tested
* You can continue publishing a single plugin package that supports both systems

---

### 📋 Summary

* Use `createFrontendPlugin` in `alpha.tsx` to define plugins in the new system
* Convert legacy pages into `PageBlueprint`
* Wrap API definitions in `ApiBlueprint`
* Register widgets and other UI components using `ComponentBlueprint`
* Use compatibility utilities to bridge old routeRefs and components

---

### 🌟 Advantages of Migrating to the New Frontend System

* **Better Performance**: Through lazy loading and code splitting
* **Federated App Ready**: Seamlessly supports micro frontend architecture
* **Simplified Plugin Structure**: Declarative APIs reduce boilerplate and improve maintainability
* **Strong Typing and Modularity**: Ensures better refactoring and tooling support
* **Compatibility Layer**: Allows gradual migration while maintaining legacy support
* **Improved Composability**: Easier to reuse and share components, pages, and APIs across plugins

---

### 🔗 References

* [Plugin Migration Guide](https://backstage.io/docs/frontend-system/building-plugins/migrating)
* [Frontend System Overview](https://backstage.io/docs/frontend-system/)
* [createApp API](https://backstage.io/docs/frontend-system/building-apps/create-app)
* [Core Compatibility API](https://backstage.io/docs/frontend-system/core-compat-api)

---

### 🙌 Thank You

Migrating your plugins to the new frontend system improves composability, lazy loading, performance, and the ability to plug into federated apps. It also sets your plugin up for long-term compatibility and growth in the Backstage ecosystem.

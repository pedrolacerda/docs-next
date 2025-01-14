---
badges:
  - breaking
---

# VNode Lifecycle Events <MigrationBadges :badges="$frontmatter.badges" />

## Overview

In Vue 2, it was possible to use events to listen for key stages in a component's lifecycle. These events had names that started with the prefix `hook:`, followed by the name of the corresponding lifecycle hook.

In Vue 3, this prefix has been changed to `vnode-`. In addition, these events are now available for HTML elements as well as components.

## 2.x Syntax

In Vue 2, the event name is the same as the equivalent lifecycle hook, prefixed with `hook:`:

```html
<template>
  <child-component @hook:updated="onUpdated">
</template>
```

## 3.x Syntax

In Vue 3, the event name is prefixed with `vnode-`:

```html
<template>
  <child-component @vnode-updated="onUpdated">
</template>
```

Or just `vnode` if you're using camelCase:

```html
<template>
  <child-component @vnodeUpdated="onUpdated">
</template>
```

## Estratégia de Migração

In most cases it should just require changing the prefix. The lifecycle hooks `beforeDestroy` and `destroyed` have been renamed to `beforeUnmount` and `unmounted` respectively, so the corresponding event names will also need to be updated.

[Sinalizadores na compilação de migração: `INSTANCE_EVENT_HOOKS`](migration-build.html#configuracao-de-compatibilidade)

## See Also

- [Migration guide - Events API](/guide/migration/events-api.html)

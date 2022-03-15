---
title: Overview
---

# Optimizer

Qwik's philosophy is to delay loading code for as long as possible. To do that, Qwik relays on Optimizer to re-arrange the code for lazy loading. The Optimizer is code level transformation that runs as part of the rollup. (Optimizer is written in Rust (and available as WASM) for instant performance)

The Optimizer looks for `$` and applies a transformation that extracts the expression following the `$` and turns it into a lazy-loadable and importable symbol.

Let's start by looking at a simple `Counter` example:

```typescript=
const Counter = component$(() => {
  const store = useStore({count: 0});
  return $(() => (
    <button on$:click={() => store.count++}>
      {store.count}
    </button>
  ));
});
```

The above code represents what a developer would write to describe the component. Below are the transformations that the Optimizer applies to the code to make the code lazy-loadable.

```typescript=
const Counter = component(qrl('./chunk-a.js', 'Counter_onMount'));
```

`chunk-a.js`:

```typescript=
export const Counter_onMount = () => {
  const store = useStore({count: 0});
  return qrl('./chunk-b.js', 'Counter_onRender', [store]);
};
```

`chunk-b.js`:

```typescript=
const Counter_onRender = () => {
  const [store] = useLexicalScope();
  return (
    <button on:click={qrl('./chunk-c.js', 'Counter_onClick', [store])}>
      {store.count}
    </button>
  );
};
```

`chunk-c.js`:

```typescript=
const Counter_onClick = () => {
  const [store] = useLexicalScope();
  return store.count++;
};
```

## Prefetching

## Runtime Based Optimization
# vaie’s headless ui components
My custom set of headless UI components (state and interaction only, you provide elements and styling through `{#snippet}`s)

## Installation
```bash
deno add npm:@vaie/hui
```

## Development
```bash
# run the documentation app
deno task dev
```

```bash
# as JSR does not support Svelte libraries, use PNPM to publish to NPM instead:
pnpm login
pnpm publish
```
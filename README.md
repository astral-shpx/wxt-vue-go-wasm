# WXT + Vue 3 + Go

WXT template + WASM with Go.

This template should help get you started developing with Vue 3 in WXT.

## Recommended IDE Setup

- [VS Code](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

# Instructions starting from scratch

First install [Go](https://go.dev/doc/install), [tinygo](https://tinygo.org/getting-started/install/) and binaryen

```sh
scoop install go
scoop install tinygo
scoop install binaryen
```

Then [install pnpm](https://pnpm.io/installation) and bootstrap WXT with your favourite frontend framework. I will be using Vue

```sh
pnpm dlx wxt@latest init <project-name>
```

Create directory for the Go code and make a Go source file

```sh
mkdir wasm_code
cd wasm_code
touch add.go
```

Paste the following in `add.go`

```go
package main

// The line below is very important
//export Add
func Add(a int, b int) int {
	return a + b
}

func main() {
}
```

Compile your Go code

```sh
tinygo build -o ../assets/add.wasm -target=wasm ./add.go
```

Open `components/HelloWorld.vue` and edit it as follows

```vue
<script lang="ts" setup>
import { ref } from "vue";
import addModule from "@/assets/add.wasm?url";

defineProps({
  msg: String,
});

const wasmInstance = ref<WasmInstance | null>(null);
const wasmResult = ref<number>(0);

const a = ref(2);
const b = ref(5);

interface WasmExports {
  Add: (a: number, b: number) => number;
  // Add other exports here
}

interface WasmInstance {
  exports: WasmExports;
}

const loadWasmModule = async () => {
  try {
    const importObject = {
      gojs: {
        Add: (a: number, b: number) => {},
      },
      wasi_snapshot_preview1: { fd_write: () => {} }, // Needed for instantiateStreaming, otherwise throws errors TypeError and LinkError
    };

    const fetchedModule = await fetch(addModule);

    const result = await WebAssembly.instantiateStreaming(
      fetchedModule,
      importObject
    );

    // console.log(result.instance.exports);

    wasmInstance.value = result.instance as unknown as WasmInstance;
  } catch (error) {
    console.error("Failed to load WebAssembly module:", error);
  }
};

const addWasm = () => {
  if (wasmInstance.value) {
    const result = wasmInstance.value.exports.Add(a.value, b.value);
    wasmResult.value = result;
  } else {
    console.error("WebAssembly module not loaded.");
  }
};

loadWasmModule();
</script>

<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <div>click below to sum {{ a }} and {{ b }}</div>
    <button type="button" @click="addWasm">result is {{ wasmResult }}</button>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>

  <p>
    Install
    <a href="https://github.com/vuejs/language-tools" target="_blank">Volar</a>
    in your IDE for a better DX
  </p>
  <p class="read-the-docs">Click on the WXT and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
```

Make sure you have `content_security_policy` set in your wxt config

For Chrome

```ts
import { defineConfig } from "wxt";

// See https://wxt.dev/api/config.html
export default defineConfig({
  modules: ["@wxt-dev/module-vue"],
  manifest: {
    content_security_policy: {
      extension_pages:
        "script-src 'self' 'wasm-unsafe-eval'; object-src 'self'",
    },
  },
});
```

For Firefox it should be

```ts
import { defineConfig } from "wxt";

// See https://wxt.dev/api/config.html
export default defineConfig({
  modules: ["@wxt-dev/module-vue"],
  manifest: {
    content_security_policy: "script-src 'self' 'wasm-eval'; object-src 'self'",
  },
});
```

PNPM install and run your extension


```sh
pnpm install
```

If you're getting a warning, install vite

```sh
pnpm install vite
```

Then

```sh
pnpm dev
```

or

```sh
pnpm dev:firefox
```

Congratulations!🎉 You are now ready to experiment with WASM and web extensions.

Additionally, check out the [WASM tinygo guide](https://tinygo.org/docs/guides/webassembly/wasm/) to see how to import functions from Javascript to Go

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

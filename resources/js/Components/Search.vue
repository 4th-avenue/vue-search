<script setup>
import { ref, onMounted, watchEffect } from 'vue'
import {
    TransitionRoot,
    TransitionChild,
    Dialog,
    DialogPanel,
    DialogTitle,
} from '@headlessui/vue';
import Meilisearch from "meilisearch";

const isOpen = ref(false);
const query = ref("");
const client = ref(null);
const results = ref(null);
const selectedHitIndex = ref(0);

function closeModal() {
    isOpen.value = false;
    query.value = "";
    results.value = null;
    selectedHitIndex.value = 0;
}
function openModal() {
    isOpen.value = true;
}

onMounted(() => {
    client.value = new Meilisearch({host: "http://localhost:7700"});
    window.addEventListener("keydown", onKeydown);
});

const search = async (query) => {
    if(query) {
        results.value = await client.value.index('posts').search(query, {limit: 8});
    }
};

watchEffect(() => {
    search(query.value);
});

function focusPreviousHits() {
    if(selectedHitIndex.value === 0) {
        selectedHitIndex.value = results.value.hits.length - 1;
    } else {
        selectedHitIndex.value--;
    }
}

function focusNextHits() {
    if(selectedHitIndex.value < results.value.hits.length - 1) {
        selectedHitIndex.value++;
    } else {
        selectedHitIndex.value = 0;
    }
}

function onHitEnter() {
    const hit = results.value.hits[selectedHitIndex.value];
    window.location = `/articles/${hit.id}`;
    closeModal();
}

const onKeydown = (e) => {
    if(isOpen.value) return;
    if((e.metaKey || e.altKey) && e.key === "k") {
        isOpen.value = true;
    }
};
</script>

<template>
    <div class="fixed inset-0 flex items-center justify-center">
        <button type="button" @click="openModal" class="rounded-md bg-indigo-600 px-4 py-2 text-sm font-medium text-white hover:bg-indigo-400">
            Open dialog
        </button>
    </div>
    <TransitionRoot appear :show="isOpen" as="template">
        <Dialog as="div" @close="closeModal" class="relative z-10">
            <TransitionChild as="template" enter="duration-300 ease-out" enter-from="opacity-0" enter-to="opacity-100" leave="duration-200 ease-in" leave-from="opacity-100" leave-to="opacity-0">
                <div class="fixed inset-0 bg-black/25" />
            </TransitionChild>

            <div class="fixed inset-0 overflow-y-auto">
                <div class="flex min-h-full items-center justify-center p-4 text-center">
                    <TransitionChild as="template" enter="duration-300 ease-out" enter-from="opacity-0 scale-95" enter-to="opacity-100 scale-100" leave="duration-200 ease-in" leave-from="opacity-100 scale-100" leave-to="opacity-0 scale-95">
                        <DialogPanel class="w-full max-w-md transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all">
                            <input type="search" v-model="query" class="w-full border-none block rounded-md" @keydown.up="focusPreviousHits()" @keydown.down="focusNextHits()" @keyup.enter="onHitEnter" />
                            <template v-if="results">
                                Found
                                <span v-text="results.estimatedTotalHits"></span> result(s)
                                <a v-for="(hit, index) in results.hits" :key="index" :href="`/articles/${hit.id}`" class="block w-full py-2 px-3 border-b border-slate-200 rounded-md" :class="{'bg-blue-300': index === selectedHitIndex}">
                                    <h1>{{ hit.title }}</h1>
                                </a>
                            </template>
                        </DialogPanel>
                    </TransitionChild>
                </div>
            </div>
        </Dialog>
    </TransitionRoot>
</template>
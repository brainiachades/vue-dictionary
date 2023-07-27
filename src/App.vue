<script setup>
import { ref, watch } from 'vue';
import { debounce } from 'lodash';

const search = ref(null)
const entries = ref([])
const errorMsg = ref(undefined)

const searchFn = (input) => {
  fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${input}`)
    .then((res) => {
      // console.log(res);
      if (!res.ok) {
        entries.value = []
        return
      }

      return res.json()
    }).then((data) => {
      // console.log(data);
      entries.value = data
    })
    .catch(err => {
      entries.value = []
      errorMsg.value = err
    })
}

const soundFn = (url) => {
  const audio = new Audio(url)
  audio.play();
}

watch(search, debounce((newTerm) => {
  searchFn(newTerm)
}, 300))
</script>

<template>
    <div class="h-screen flex flex-col justify-center items-center w-full">
      <div class="w-full">
        <h1 class="text-blue-700 text-4xl text-center font-extrabold">
          Vue Dictionary
        </h1>
        <div class="mx-5 shadow-md flex justify-center items-center">
          <div class="mt-2 w-full">
            <input type="search" v-model="search"
              class="w-full py-3 px-5 text-lg rounded-full ring-1 ring-gray-500 focus:outline-none"
              placeholder="Search meaning ...">
          </div>
        </div>
        <div class="relative overflow-y-scroll top-2 bottom-0">
          <div class="mx-5 mt-5 h-[80vh]" v-if="entries.length > 0">
            <div v-for="entry in entries" class="bg-white w-full rounded drop-shadow-xl shadow-inner p-5">
              <div class="">
                <div class="flex justify-between items-center">
                  <div class="">
                    <h2 class="text-blue-500 text-2xl font-bold">{{ entry.word }}</h2>
                    <span class="text-amber-500 font-bold">{{ entry.phonetic }}</span>
                  </div>
                  <div class="px-3 py-2 text-white font-bold bg-blue-700 rounded"
                    @click="soundFn(entry?.phonetics[0]?.audio || entry?.phonetics[1]?.audio)">
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-6 h-6">
                      <path fill-rule="evenodd"
                        d="M4.5 5.653c0-1.426 1.529-2.33 2.779-1.643l11.54 6.348c1.295.712 1.295 2.573 0 3.285L7.28 19.991c-1.25.687-2.779-.217-2.779-1.643V5.653z"
                        clip-rule="evenodd" />
                    </svg>
                  </div>
                </div>

                <div v-for="meaning in entry.meanings" class="mt-5">
                  <p class="text-gray-700 italic">{{ meaning.partOfSpeech }}</p>
                  <ul class="font-semibold">
                    <li v-for="definition in meaning.definitions">- {{ definition.definition }}</li>
                  </ul>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<style scoped></style>

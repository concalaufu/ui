<script setup>
const people = ['Wade Cooper', 'Arlene Mccoy', 'Devon Webb', 'Tom Cook', 'Tanya Fox', 'Hellen Schmidt', 'Caroline Schultz', 'Mason Heaney', 'Claudie Smitham', 'Emil Schaefer']

const selected = ref(people[3])
</script>

<template>
  <USelectMenu v-slot="{ open }" v-model="selected" :options="people">
    <UButton>
      {{ selected }}

      <UIcon name="i-heroicons-chevron-right-20-solid" class="w-5 h-5 transition-transform" :class="[open && 'transform rotate-90']" />
    </UButton>
  </USelectMenu>
</template>

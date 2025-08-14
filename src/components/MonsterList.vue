<template>
  <div class="monster-list">
    <div v-for="m in items" :key="m.id" class="monster">
      <img :src="m.img" alt="img">
      <img src="https://static.wikia.nocookie.net/rickandmorty/images/d/db/PM-234.png/revision/latest?cb=20210627181945" alt="">
      <!-- <p>{{ m.img }}</p> -->
      <div style="flex:1">
        <div class="name">{{ m.name }} <span class="badge">VEL {{ m.spd }}</span></div>
        <div class="stat">Vida: {{ m.hp }} &nbsp; Ataque: {{ m.atk }} &nbsp; Defensa: {{ m.def }}</div>
        <div class="stack" style="margin-top:8px">
          <button class="btn secondary" @click="$emit('edit', m.id)">Editar</button>
          <button class="btn danger" @click="$emit('remove', m.id)">Eliminar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
    defineProps({
        items: { type: Array, default: () => [] }
    })

    defineEmits(['edit','remove'])
</script>

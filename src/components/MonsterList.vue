<template>
  <div class="monster-list">
    <div v-for="m in items" :key="m.id" class="monster">
      <img :src="m.img" alt="img">
      <div style="flex:1">
        <div class="name">{{ m.name }} <span class="badge">VEL {{ m.spd }}</span></div>
        <div class="stat">Vida: {{ m.hp }} &nbsp; Ataque: {{ m.atk }} &nbsp; Defensa: {{ m.def }}</div>
        <div class="stack" style="margin-top:8px">
          <button class="btn secondary" @click="$emit('edit', m.id)">Editar</button>
          <button class="btn danger" @click="$emit('remove', m.id)">
            <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
            width="24"
            height="24"
            fill="white"
            >
            <path d="M3 6h18v2H3V6zm2 3h14l-1.5 12.5c-.1.8-.7 1.5-1.5 1.5H8c-.8 0-1.4-.7-1.5-1.5L5 9zm5 2v8h2v-8H10zm4 0v8h2v-8h-2zM9 4V3h6v1h5v2H4V4h5z"/>
            </svg>
          </button>
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

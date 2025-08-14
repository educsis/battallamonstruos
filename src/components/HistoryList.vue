<template>
  <div>
    <div v-for="b in reversed" :key="b.at" class="monster">
      <div style="font-size:28px">🗓️</div>
      <div style="flex:1">
        <div class="name">{{ new Date(b.at).toLocaleString() }}</div>
        <div>🏆 <span class="winner"><strong>{{ b.winner.name }}</strong></span> venció a <span class="loser"><strong>{{ b.loser.name }}</strong></span></div>
      </div>
      <div class="stack">
        <button class="btn ghost" @click="$emit('view', b)">Ver</button>
        <button class="btn danger" @click="$emit('remove', b.at)">Eliminar</button>
      </div>
    </div>
  </div>
</template>

<script setup>
    import { computed } from 'vue'

    const props = defineProps({ battles: { type: Array, default: () => [] } })

    defineEmits(['view','remove'])
    
    const reversed = computed(() => [...props.battles].reverse())
</script>

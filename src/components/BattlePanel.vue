<template>
  <div>
    <div class="row">
      <div>
        <label>Atacante</label>
        <select v-model="idA">
          <option v-for="m in monsters" :key="m.id" :value="m.id">{{ m.name }}</option>
        </select>
      </div>
      <div>
        <label>Defensor</label>
        <select v-model="idB">
          <option v-for="m in monsters" :key="m.id" :value="m.id">{{ m.name }}</option>
        </select>
      </div>
    </div>
    <div class="stack" style="margin-top:12px">
      <button class="btn" @click="fight">Simular Batalla</button>
    </div>

    <div v-if="battle" class="timeline" style="margin-top:10px">
      <div class="turn">
        <div class="hit">Resultado:</div>
        <div class="winner">Ganador: <strong>{{ battle.winner.name }}</strong></div>
        <div>—</div>
        <div class="loser">Perdedor: <strong>{{ battle.loser.name }}</strong></div>
      </div>
      <div v-for="(t, i) in battle.turns" :key="i" class="turn">
        <div><strong>{{ t.from }}</strong> golpea a <strong>{{ t.to }}</strong></div>
        <div class="hit">-{{ t.dmg }} Hit</div>
        <div class="muted">Vida de {{ t.to }}: {{ t.toHp }}</div>
      </div>
    </div>

    <div v-else class="empty">Aún no hay batalla. Selecciona dos monstruos y presiona "Simular Batalla".</div>
  </div>
</template>

<script setup>
    import { ref, watch } from 'vue'

    const props = defineProps({
      monsters: { type: Array, default: () => [] },
      battle: { type: Object, default: null }
    })
    const emit = defineEmits(['fight'])

    const idA = ref('')
    const idB = ref('')

    watch(() => props.monsters, (list) => {
    // Llenar los selects por defecto
    if(list && list.length > 0){
        if(!idA.value) idA.value = list[0].id
        if(!idB.value) idB.value = list.length > 1 ? list[1].id : list[0].id
    } else { idA.value = ''; idB.value = '' }
    }, { immediate: true, deep: true })

    function fight(){
    emit('fight', { idA: idA.value, idB: idB.value })
    }
</script>

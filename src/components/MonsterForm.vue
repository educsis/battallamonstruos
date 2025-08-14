<template>
  <form @submit.prevent="onSubmit">
    <input type="hidden" :value="local.id" />
    <label>Nombre</label>
    <input type="text" v-model="local.name" placeholder="Ejemplo: Cthulhu Monster">

    <div class="row">
      <div>
        <label>Vida</label>
        <input type="number" v-model.number="local.hp" min="1">
      </div>
      <div>
        <label>Ataque</label>
        <input type="number" v-model.number="local.atk" min="1">
      </div>
    </div>

    <div class="row">
      <div>
        <label>Defensa</label>
        <input type="number" v-model.number="local.def" min="0">
      </div>
      <div>
        <label>Velocidad</label>
        <input type="number" v-model.number="local.spd" min="1">
      </div>
    </div>

    <label>URL de Imagen</label>
    <input type="text" v-model="local.img" placeholder="https://...">

    <div class="stack" style="margin-top:12px">
      <button class="btn" type="submit">{{ isEdit ? 'Actualizar' : 'Guardar' }}</button>
      <button class="btn ghost" type="button" @click="$emit('cancel')">Cancelar</button>
    </div>
  </form>
</template>

<script setup>
    import { reactive, watch } from 'vue'

    const props = defineProps({
        model: { type: Object, required: true },
        isEdit: { type: Boolean, default: false }
    })

    const emit = defineEmits(['save','cancel'])

    // Copia local para no mutar el objeto original hasta guardar
    const local = reactive({ id:'', name:'', hp:10, atk:5, def:2, spd:3, img:'' })

    watch(() => props.model, (val) => {
    Object.assign(local, val || { id:'', name:'', hp:10, atk:5, def:2, spd:3, img:'' })
    }, { deep: true, immediate: true })

    function onSubmit(){
        emit('save', { ...local })
    }
</script>

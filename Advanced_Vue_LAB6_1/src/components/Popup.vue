<template>
  <Teleport to="body">
    <div v-if="modelValue" class="popup-overlay" @click.self="close">
      <div class="popup-container">
        <button class="popup-close" @click="close">✖</button>
        <div class="popup-content">
          <slot></slot>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])
const close = () => emit('update:modelValue', false)
</script>

<style scoped>
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
.popup-container {
  background: white;
  border-radius: 12px;
  padding: 20px;
  min-width: 280px;
  max-width: 90%;
  position: relative;
  box-shadow: 0 4px 20px rgba(0,0,0,0.2);
}
.popup-close {
  position: absolute;
  top: 8px;
  right: 12px;
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
}
.popup-content {
  margin-top: 16px;
}
</style>
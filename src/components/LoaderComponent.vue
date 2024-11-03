<template>
  <div class="divLoader">
    <span class="loader"></span>
    <small class="loaderMessage">{{ props.message }}</small>
  </div>
</template>

<script setup>
  import { defineProps } from 'vue';

  const props = defineProps({
    message: {
      type: String,
      default: 'Por favor espere...',
    },
  });
</script>

<style  scoped>

.divLoader {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 1rem;
  position: fixed;	
  background-color: rgba(0, 0, 0, 0.493);
  z-index: 1000;
}

.loaderMessage {
  font-size: large;
}

.loader {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  display: inline-block;
  border-top: 4px solid #FFF;
  border-right: 4px solid transparent;
  box-sizing: border-box;
  animation: rotation 1s linear infinite;
}
.loader::after {
  content: '';  
  box-sizing: border-box;
  position: absolute;
  left: 0;
  top: 0;
  width: 48px;
  height: 48px;
  border-radius: 50%;
  border-left: 4px solid #FF3D00;
  border-bottom: 4px solid transparent;
  animation: rotation 0.5s linear infinite reverse;
}
@keyframes rotation {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
} 
</style>
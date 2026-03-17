<template>
  <div class="app-container">
    <header class="app-header">
      <h1>📦 PO XML Builder</h1>
      <p>Overseas Shipping - Advanced Shipment Notification</p>
    </header>
    <POBuilder />
  </div>
</template>

<script setup>
import POBuilder from './components/POBuilder.vue'
</script>

<style scoped>
.app-container {
  max-width: 1200px;
  margin: 0 auto;
}

.app-header {
  background: white;
  padding: 40px 30px;
  border-radius: 12px 12px 0 0;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 0;
}

.app-header h1 {
  color: #667eea;
  font-size: 2.5em;
  margin-bottom: 10px;
}

.app-header p {
  color: #999;
  font-size: 1em;
}
</style>

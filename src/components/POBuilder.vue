<template>
  <div class="po-builder">
    <!-- Envelope Section -->
    <section class="form-section">
      <h2>📋 Shipment Header</h2>
      <div class="form-grid">
        <div class="form-group">
          <label>Sender ID</label>
          <input v-model="form.senderID" type="text" placeholder="e.g., KN">
        </div>
        <div class="form-group">
          <label>Receiver ID</label>
          <input v-model="form.receiverID" type="text" placeholder="e.g., DREAMS">
        </div>
        <div class="form-group">
          <label>Message Date</label>
          <input v-model="form.messageDate" type="date">
        </div>
        <div class="form-group">
          <label>Message Time</label>
          <input v-model="form.messageTime" type="time">
        </div>
        <div class="form-group">
          <label>Envelope ID</label>
          <input v-model="form.envelopeId" type="text" placeholder="e.g., V00551">
        </div>
        <div class="form-group">
          <label>Purchase Order Number</label>
          <input v-model="form.purchaseOrderNumber" type="text" placeholder="e.g., DRMS-PO-000000551">
        </div>
      </div>
    </section>

    <!-- Status & Vessel Section -->
    <section class="form-section">
      <h2>🚢 Vessel & Shipping Info</h2>
      <div class="form-grid">
        <div class="form-group">
          <label>Status Code</label>
          <input v-model="form.statusCode" type="text" placeholder="e.g., 1300">
        </div>
        <div class="form-group">
          <label>Status Description</label>
          <input v-model="form.statusDescription" type="text" placeholder="e.g., SHIPPED">
        </div>
        <div class="form-group">
          <label>Status Date</label>
          <input v-model="form.statusDate" type="date">
        </div>
        <div class="form-group">
          <label>Status Time</label>
          <input v-model="form.statusTime" type="time">
        </div>
      </div>

      <div class="form-grid">
        <div class="form-group">
          <label>Vessel Name</label>
          <input v-model="form.vesselName" type="text" placeholder="e.g., CMA CGM BOUGAINVILLE">
        </div>
        <div class="form-group">
          <label>ETS (Expected Time of Sailing)</label>
          <input v-model="form.ets" type="date">
        </div>
        <div class="form-group">
          <label>ETA (Expected Time of Arrival)</label>
          <input v-model="form.eta" type="date">
        </div>
        <div class="form-group">
          <label>Port of Loading (POL)</label>
          <input v-model="form.pol" type="text" placeholder="e.g., CNSHA">
        </div>
        <div class="form-group">
          <label>Port of Discharge (POD)</label>
          <input v-model="form.pod" type="text" placeholder="e.g., GBFXT">
        </div>
      </div>

      <div class="form-grid">
        <div class="form-group">
          <label>Container Number</label>
          <input v-model="form.containerNumber" type="text" placeholder="e.g., CMAU924">
        </div>
        <div class="form-group">
          <label>Container Type</label>
          <select v-model="form.containerType">
            <option value="">Select Type</option>
            <option value="20DC">20DC (20ft Dry Container)</option>
            <option value="40DC">40DC (40ft Dry Container)</option>
            <option value="40HC">40HC (40ft High Cube)</option>
            <option value="20HC">20HC (20ft High Cube)</option>
          </select>
        </div>
      </div>
    </section>

    <!-- Line Items Section -->
    <section class="form-section">
      <div class="section-header">
        <h2>📦 Line Items</h2>
        <button class="btn-add" @click="addLineItem">+ Add Line Item</button>
      </div>

      <div v-if="form.lineItems.length === 0" class="empty-state">
        <p>No line items yet. Click "Add Line Item" to start.</p>
      </div>

      <div v-for="(item, index) in form.lineItems" :key="index" class="line-item">
        <div class="line-item-header">
          <h3>Line #{{ index + 1 }}</h3>
          <button class="btn-remove" @click="removeLineItem(index)">Remove</button>
        </div>

        <div class="form-grid">
          <div class="form-group">
            <label>Product Number *</label>
            <input v-model="item.productNumber" type="text" placeholder="e.g., 733-00077" required>
          </div>
          <div class="form-group">
            <label>Ordered Quantity *</label>
            <input v-model.number="item.orderedQuantity" type="number" placeholder="e.g., 1000" required>
          </div>
          <div class="form-group">
            <label>Actual Quantity *</label>
            <input v-model.number="item.actualQuantity" type="number" placeholder="e.g., 1000" required>
          </div>
        </div>
      </div>
    </section>

    <!-- Actions -->
    <section class="form-actions">
      <button class="btn-primary" @click="generateXML">Generate XML</button>
      <button class="btn-secondary" @click="downloadXML" v-if="xmlOutput">Download XML</button>
      <button class="btn-secondary" @click="resetForm">Reset Form</button>
    </section>

    <!-- XML Preview -->
    <section v-if="xmlOutput" class="xml-preview-section">
      <h2>📄 XML Output</h2>
      <div class="xml-preview">
        <pre>{{ xmlOutput }}</pre>
        <button class="btn-copy" @click="copyToClipboard">Copy to Clipboard</button>
      </div>
    </section>

    <!-- Toast Notification -->
    <div v-if="toast.show" :class="['toast', toast.type]">
      {{ toast.message }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const form = ref({
  senderID: 'KN',
  receiverID: 'DREAMS',
  messageDate: new Date().toISOString().split('T')[0],
  messageTime: '12:00',
  envelopeId: '',
  purchaseOrderNumber: '',
  statusCode: '1300',
  statusDescription: 'SHIPPED',
  statusDate: new Date().toISOString().split('T')[0],
  statusTime: '00:00',
  vesselName: '',
  ets: new Date().toISOString().split('T')[0],
  eta: new Date().toISOString().split('T')[0],
  pol: 'CNSHA',
  pod: 'GBFXT',
  containerNumber: '',
  containerType: '40HC',
  lineItems: []
})

const xmlOutput = ref('')
const toast = ref({ show: false, message: '', type: 'success' })

function formatDate(dateStr) {
  if (!dateStr) return ''
  const [year, month, day] = dateStr.split('-')
  return `${day}-${month}-${year}`
}

function addLineItem() {
  form.value.lineItems.push({
    productNumber: '',
    orderedQuantity: '',
    actualQuantity: ''
  })
}

function removeLineItem(index) {
  form.value.lineItems.splice(index, 1)
}

function generateXML() {
  // Validation
  if (!form.value.envelopeId.trim()) {
    showToast('Envelope ID is required', 'error')
    return
  }
  if (!form.value.purchaseOrderNumber.trim()) {
    showToast('Purchase Order Number is required', 'error')
    return
  }
  if (form.value.lineItems.length === 0) {
    showToast('At least one line item is required', 'error')
    return
  }

  // Check all line items are complete
  for (let item of form.value.lineItems) {
    if (!item.productNumber.trim() || !item.orderedQuantity || !item.actualQuantity) {
      showToast('All line items must have Product Number and quantities', 'error')
      return
    }
  }

  // Generate XML
  let xml = '<?xml version="1.0" encoding="utf-8"?>\n'
  xml += '<ASN>\n'

  // Envelope
  xml += '<Envelope>\n'
  xml += `<SenderID>${escapeXml(form.value.senderID)}</SenderID>\n`
  xml += `<ReceiverID>${escapeXml(form.value.receiverID)}</ReceiverID>\n`
  xml += `<MessageDate>${formatDate(form.value.messageDate)}</MessageDate>\n`
  xml += `<MessageTime>${form.value.messageTime}</MessageTime>\n`
  xml += `<EnvelopeId>${escapeXml(form.value.envelopeId)}</EnvelopeId>\n`
  xml += '</Envelope>\n'

  // Event
  xml += '<Event>\n'

  // Status
  xml += '<Status>\n'
  xml += `<Code>${escapeXml(form.value.statusCode)}</Code>\n`
  xml += `<Description>${escapeXml(form.value.statusDescription)}</Description>\n`
  xml += `<Date>${formatDate(form.value.statusDate)}</Date>\n`
  xml += `<Time>${form.value.statusTime}</Time>\n`
  xml += '</Status>\n'

  // Vessel
  xml += '<Vessel>\n'
  xml += `<Name>${escapeXml(form.value.vesselName)}</Name>\n`
  xml += `<ETS>${formatDate(form.value.ets)}</ETS>\n`
  xml += `<ETA>${formatDate(form.value.eta)}</ETA>\n`
  xml += `<POL>${escapeXml(form.value.pol)}</POL>\n`
  xml += `<POD>${escapeXml(form.value.pod)}</POD>\n`
  xml += '</Vessel>\n'

  // Details (Line Items)
  for (let i = 0; i < form.value.lineItems.length; i++) {
    const item = form.value.lineItems[i]
    xml += '<Details>\n'
    xml += `<PurchaseOrderNumber>${escapeXml(form.value.purchaseOrderNumber)}</PurchaseOrderNumber>\n`
    xml += `<ProductNumber>${escapeXml(item.productNumber)}</ProductNumber>\n`
    xml += `<LineItemNumber>${i + 1}</LineItemNumber>\n`
    xml += `<OrderedQuantity>${item.orderedQuantity}</OrderedQuantity>\n`
    xml += '<Equipment>\n'
    xml += `<ContainerNumber>${escapeXml(form.value.containerNumber)}</ContainerNumber>\n`
    xml += `<ContainerType>${escapeXml(form.value.containerType)}</ContainerType>\n`
    xml += `<ActualQuantity>${item.actualQuantity}</ActualQuantity>\n`
    xml += '</Equipment>\n'
    xml += '</Details>\n'
  }

  xml += '</Event>\n'
  xml += '</ASN>'

  xmlOutput.value = xml
  showToast('XML generated successfully!', 'success')
}

function escapeXml(str) {
  if (!str) return ''
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&apos;')
}

function downloadXML() {
  const element = document.createElement('a')
  const file = new Blob([xmlOutput.value], { type: 'application/xml' })
  element.href = URL.createObjectURL(file)
  element.download = `PO_${form.value.envelopeId}_${Date.now()}.xml`
  document.body.appendChild(element)
  element.click()
  document.body.removeChild(element)
  showToast('XML downloaded!', 'success')
}

function copyToClipboard() {
  navigator.clipboard.writeText(xmlOutput.value)
  showToast('Copied to clipboard!', 'success')
}

function resetForm() {
  form.value.lineItems = []
  xmlOutput.value = ''
  showToast('Form reset', 'success')
}

function showToast(message, type = 'success') {
  toast.value = { show: true, message, type }
  setTimeout(() => {
    toast.value.show = false
  }, 3000)
}
</script>

<style scoped>
.po-builder {
  background: white;
  border-radius: 0 0 12px 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 30px;
  margin-bottom: 20px;
}

.form-section {
  margin-bottom: 40px;
  padding-bottom: 30px;
  border-bottom: 1px solid #e5e7eb;
}

.form-section:last-of-type {
  border-bottom: none;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.form-section h2 {
  color: #333;
  font-size: 1.5em;
  margin-bottom: 20px;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  margin-bottom: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
}

.form-group label {
  font-weight: 600;
  color: #555;
  margin-bottom: 8px;
  font-size: 0.95em;
}

.form-group input,
.form-group select {
  padding: 12px 14px;
  border: 2px solid #e5e7eb;
  border-radius: 6px;
  font-size: 1em;
  transition: all 0.3s ease;
  font-family: inherit;
}

.form-group input:focus,
.form-group select:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.line-item {
  background: #f9fafb;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
}

.line-item-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.line-item-header h3 {
  color: #333;
  font-size: 1.1em;
}

.empty-state {
  text-align: center;
  padding: 40px 20px;
  color: #999;
  background: #f9fafb;
  border-radius: 8px;
  border: 2px dashed #e5e7eb;
}

/* Buttons */
.btn-add {
  background: #667eea;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.95em;
}

.btn-add:hover {
  background: #5568d3;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.btn-remove {
  background: #ef4444;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9em;
}

.btn-remove:hover {
  background: #dc2626;
  transform: translateY(-2px);
}

.form-actions {
  display: flex;
  gap: 12px;
  margin-top: 30px;
  flex-wrap: wrap;
}

.btn-primary,
.btn-secondary {
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 1em;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

.btn-secondary {
  background: #e5e7eb;
  color: #333;
}

.btn-secondary:hover {
  background: #d1d5db;
  transform: translateY(-2px);
}

.xml-preview-section {
  margin-top: 40px;
  padding-top: 30px;
  border-top: 2px solid #e5e7eb;
}

.xml-preview-section h2 {
  color: #333;
  font-size: 1.5em;
  margin-bottom: 20px;
}

.xml-preview {
  background: #1f2937;
  color: #e5e7eb;
  padding: 20px;
  border-radius: 8px;
  overflow-x: auto;
  position: relative;
}

.xml-preview pre {
  margin: 0;
  font-family: 'Courier New', monospace;
  font-size: 0.9em;
  line-height: 1.6;
}

.btn-copy {
  position: absolute;
  top: 15px;
  right: 15px;
  background: #667eea;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  font-size: 0.9em;
  transition: all 0.3s ease;
}

.btn-copy:hover {
  background: #5568d3;
  transform: translateY(-2px);
}

/* Toast Notification */
.toast {
  position: fixed;
  bottom: 20px;
  right: 20px;
  padding: 16px 24px;
  border-radius: 8px;
  font-weight: 600;
  animation: slideIn 0.3s ease;
  z-index: 1000;
}

.toast.success {
  background: #10b981;
  color: white;
}

.toast.error {
  background: #ef4444;
  color: white;
}

@keyframes slideIn {
  from {
    transform: translateX(400px);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@media (max-width: 768px) {
  .po-builder {
    padding: 20px;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }

  .section-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .btn-add {
    align-self: flex-start;
  }

  .form-actions {
    flex-direction: column;
  }

  .form-actions button {
    width: 100%;
  }
}
</style>

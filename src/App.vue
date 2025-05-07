<template>
  <div id="invoice" class="max-w-3xl mx-auto p-6 bg-white shadow-xl rounded-2xl border border-gray-200">
    <!-- Invoice Header -->
    <div class="text-center mb-8">
      <h1 class="text-3xl font-bold text-blue-600">Invoice for {{ previousMonth }} </h1>
      <p class="text-gray-500">Invoice Date: {{ today }}</p>
    </div>

    <!-- Personal & Client Info -->
    <div class="grid grid-cols-2 gap-6 text-sm text-gray-700">
      <div>
        <h2 class="font-semibold text-blue-500">From:</h2>
        <p class="font-bold text-xl">{{ sender.name }}</p>
        <p>{{ sender.address }}</p>
        <p>Phone: {{ sender.phone }}</p>
        <p>Email: {{ sender.email }}</p>
      </div>
      <div>
        <h2 class="font-semibold text-blue-500">To:</h2>
        <p class="font-bold text-teal-600 text-2xl">{{ client.name }}</p>
        <p>{{ client.address }}</p>
        <p>Phone: {{ client.phone }}</p>
        <p>Email: {{ client.email }}</p>
      </div>
    </div>

    <!-- Input Section -->
    <div class="my-6 grid grid-cols-2 gap-4">
      <div>
        <label class="block text-sm font-medium text-gray-600">Total Work Hours</label>
        <input v-model="inputHours" type="text" class="mt-1 w-full border rounded px-3 py-2" placeholder="e.g., 17:52" />
      </div>
      <div>
        <label class="block text-sm font-medium text-gray-600">Hourly Rate (Tk)</label>
        <input v-model.number="rate" type="number" class="mt-1 w-full border rounded px-3 py-2" />
      </div>
    </div>

    <!-- Invoice Summary -->
    <div class="border-t pt-4 mt-4 text-gray-700">
      <div class="flex justify-between mb-2">
        <span>Work Hours</span>
        <span>{{ displayHours }}</span>
      </div>
      <div class="flex justify-between mb-2">
        <span>Hourly Rate</span>
        <span>Tk {{ rate }}</span>
      </div>
      <div class="flex justify-between font-semibold text-lg border-t pt-2">
        <span>Total Amount</span>
        <span>Tk {{ totalAmount.toLocaleString() }}</span>
      </div>
      <div class="mt-2 text-sm italic text-gray-500">
        Total in words: {{ amountInWords }} Taka Only
      </div>
    </div>

    

    <!-- Thank You Message -->
    <div class="mt-8 text-center text-sm text-gray-600">
      <p>Thank you for the opportunity to work with {{ client.name }}.</p>
      <p>I appreciate your trust and support, and look forward to our continued collaboration.</p>
    </div>
  </div>

  <!-- Export Button -->
    <div class="mt-6 text-center">
      <button @click="exportPDF" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">Download PDF</button>
    </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import html2pdf from 'html2pdf.js'

const today = new Date().toLocaleDateString('en-US', {
  day: 'numeric',
  month: 'long',
  year: 'numeric'
}).replace(/(\w+)\s(\d+),\s(\d+)/, '$2 $1, $3')
const current = new Date()
const previousMonth = new Date(current.setMonth(current.getMonth() - 1)).toLocaleString('default', { month: 'long', year: 'numeric' })

const sender = ref({
  name: 'Abdur Razzaque (Hassan)',
  address: 'Paler Baliatoli, M. Baliatoli, PorirKhal, Barguna Sadar, Barguna-8700',
  phone: '01813678273',
  email: 'mdarh411@gmail.com',
})
const client = ref({
  name: 'Troubleshoot Ltd.',
  address: 'Concord Modhumoti Plaza, Plot # 11 (4th Floor), Road # 11, Block # G, Banani, Dhaka-1213',
  phone: '+880 9677-889900',
  email: 'contact@troubleshoot.com.bd',
})
const inputHours = ref('17:52')
const rate = ref(400)

const displayHours = computed(() => {
  const [h, m] = inputHours.value.split(':').map(Number)
  return `${h}h ${m}m`
})

const totalAmount = computed(() => {
  const [h, m] = inputHours.value.split(':').map(Number)
  const totalMinutes = h * 60 + m
  return Math.round((totalMinutes / 60) * rate.value)
})

function numberToWords(n) {
  const a = ["", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine", "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"]
  const b = ["", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"]
  if (n < 20) return a[n]
  if (n < 100) return b[Math.floor(n / 10)] + (n % 10 ? " " + a[n % 10] : "")
  if (n < 1000) return a[Math.floor(n / 100)] + " Hundred" + (n % 100 ? " and " + numberToWords(n % 100) : "")
  if (n < 100000) return numberToWords(Math.floor(n / 1000)) + " Thousand" + (n % 1000 ? " " + numberToWords(n % 1000) : "")
  return ""
}

const amountInWords = computed(() => numberToWords(totalAmount.value))

function exportPDF() {
  const el = document.getElementById('invoice')
  const opt = {
    margin: 1,
    filename: `${previousMonth.replace(/ /g, '_')}_Invoice.pdf`,
    image: { type: 'jpeg', quality: 1 },
    html2canvas: { 
      scale: 2,
      useCORS: true,
      logging: true
    },
    jsPDF: { 
      unit: 'in', 
      format: 'a4', 
      orientation: 'portrait'
    }
  }
  
  // Remove the original download button temporarily
  const downloadBtn = document.querySelector('.fixed.top-4.right-4')
  if (downloadBtn) downloadBtn.style.display = 'none'
  
  html2pdf().set(opt).from(el).save().then(() => {
    // Show the button again after PDF is generated
    if (downloadBtn) downloadBtn.style.display = 'block'
  })
}

onMounted(() => {
  const savedSender = localStorage.getItem('invoice_sender')
  const savedClient = localStorage.getItem('invoice_client')
  if (savedSender) sender.value = JSON.parse(savedSender)
  if (savedClient) client.value = JSON.parse(savedClient)
})

watch(sender, (val) => localStorage.setItem('invoice_sender', JSON.stringify(val)), { deep: true })
watch(client, (val) => localStorage.setItem('invoice_client', JSON.stringify(val)), { deep: true })
</script>

<style>
body {
  background-color: #f8fafc;
}
</style>

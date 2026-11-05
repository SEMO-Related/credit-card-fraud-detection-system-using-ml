<template>
  <div class="min-h-screen bg-slate-950 text-slate-200 p-6 md:p-12 font-sans selection:bg-indigo-500/30">
    
    <!-- Header -->
    <header ref="headerRef" class="flex flex-col md:flex-row justify-between items-start md:items-center mb-10 opacity-0 translate-y-4">
      <div>
        <h1 class="text-3xl font-bold text-white tracking-tight">FraudLens Engine</h1>
        <p class="text-slate-400 mt-1">Real-time ML Transaction Monitoring</p>
      </div>
      <div class="mt-4 md:mt-0 flex gap-4">
        <button class="px-5 py-2.5 rounded-lg border border-slate-700 bg-slate-900 hover:bg-slate-800 transition-colors text-sm font-medium">
          Export Report
        </button>
        <button class="px-5 py-2.5 rounded-lg bg-indigo-600 hover:bg-indigo-500 transition-colors text-white text-sm font-medium flex items-center gap-2 shadow-lg shadow-indigo-500/20">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"></path></svg>
          Upload Dataset
        </button>
      </div>
    </header>

    <!-- KPI Cards -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-10">
      <div v-for="(stat, index) in stats" :key="index" class="kpi-card opacity-0 translate-y-4 bg-slate-900 border border-slate-800 rounded-xl p-6 shadow-sm">
        <h3 class="text-slate-400 text-sm font-medium mb-2">{{ stat.label }}</h3>
        <div class="flex items-end gap-3">
          <span class="text-3xl font-bold text-white">{{ stat.value }}</span>
          <span :class="stat.trend > 0 ? 'text-emerald-400' : 'text-rose-400'" class="text-sm font-medium mb-1">
            {{ stat.trend > 0 ? '+' : '' }}{{ stat.trend }}%
          </span>
        </div>
      </div>
    </div>

    <!-- Main Grid -->
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      
      <!-- Chart Section -->
      <div ref="chartContainerRef" class="lg:col-span-1 bg-slate-900 border border-slate-800 rounded-xl p-6 opacity-0 translate-y-4 shadow-sm flex flex-col">
        <h2 class="text-lg font-semibold text-white mb-6">Risk Distribution</h2>
        <div class="flex-grow flex items-center justify-center min-h-[250px]">
          <Doughnut :data="chartData" :options="chartOptions" />
        </div>
      </div>

      <!-- Data Table Section -->
      <div class="lg:col-span-2 bg-slate-900 border border-slate-800 rounded-xl shadow-sm overflow-hidden flex flex-col">
        <div ref="tableHeaderRef" class="p-6 border-b border-slate-800 opacity-0">
          <h2 class="text-lg font-semibold text-white">Recent Transactions</h2>
        </div>
        <div class="overflow-x-auto">
          <table class="w-full text-left text-sm whitespace-nowrap">
            <thead class="bg-slate-950/50 text-slate-400">
              <tr>
                <th class="px-6 py-4 font-medium">Transaction ID</th>
                <th class="px-6 py-4 font-medium">Date & Time</th>
                <th class="px-6 py-4 font-medium">Amount</th>
                <th class="px-6 py-4 font-medium">ML Confidence</th>
                <th class="px-6 py-4 font-medium">Status</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="tx in dummyTransactions" :key="tx.id" class="table-row opacity-0 border-b border-slate-800/50 hover:bg-slate-800/50 transition-colors">
                <td class="px-6 py-4 font-mono text-slate-300">{{ tx.id }}</td>
                <td class="px-6 py-4 text-slate-400">{{ tx.date }}</td>
                <td class="px-6 py-4 text-white font-medium">${{ tx.amount.toFixed(2) }}</td>
                <td class="px-6 py-4">
                  <div class="flex items-center gap-2">
                    <div class="w-16 h-2 bg-slate-800 rounded-full overflow-hidden">
                      <div class="h-full rounded-full" :class="tx.isFraud ? 'bg-rose-500' : 'bg-emerald-500'" :style="{ width: `${tx.confidence}%` }"></div>
                    </div>
                    <span class="text-xs text-slate-400">{{ tx.confidence }}%</span>
                  </div>
                </td>
                <td class="px-6 py-4">
                  <span v-if="tx.isFraud" class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-rose-500/10 text-rose-400 border border-rose-500/20 shadow-[0_0_10px_rgba(244,63,94,0.2)]">
                    Fraud Detected
                  </span>
                  <span v-else class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-emerald-500/10 text-emerald-400 border border-emerald-500/20">
                    Genuine
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import gsap from 'gsap'
import { Doughnut } from 'vue-chartjs'
import { Chart as ChartJS, ArcElement, Tooltip, Legend } from 'chart.js'

ChartJS.register(ArcElement, Tooltip, Legend)

// Template Refs for GSAP
const headerRef = ref(null)
const chartContainerRef = ref(null)
const tableHeaderRef = ref(null)

// Dummy KPI Data
const stats = ref([
  { label: 'Total Analyzed', value: '24,592', trend: 12.5 },
  { label: 'Flagged Fraudulent', value: '143', trend: -4.2 },
  { label: 'Model Precision (F1)', value: '0.94', trend: 2.1 },
])

// Dummy Transaction Data
const dummyTransactions = ref([
  { id: 'TXN-8X92-AL41', date: '2026-11-05 14:22:01', amount: 1250.00, confidence: 98, isFraud: true },
  { id: 'TXN-3P11-ZM99', date: '2026-11-05 14:18:44', amount: 45.20, confidence: 99, isFraud: false },
  { id: 'TXN-7L88-QW22', date: '2026-11-05 14:15:12', amount: 399.99, confidence: 95, isFraud: false },
  { id: 'TXN-9M44-JK10', date: '2026-11-05 13:58:30', amount: 8900.50, confidence: 91, isFraud: true },
  { id: 'TXN-2B33-TY77', date: '2026-11-05 13:42:19', amount: 12.99, confidence: 99, isFraud: false },
  { id: 'TXN-5V66-XC44', date: '2026-11-05 13:30:05', amount: 450.00, confidence: 88, isFraud: false },
])

// Chart Configuration
const chartData = {
  labels: ['Genuine', 'Fraudulent'],
  datasets: [
    {
      backgroundColor: ['#10B981', '#F43F5E'], // Tailwind Emerald-500, Rose-500
      borderColor: ['#020617', '#020617'], // Matches bg-slate-950
      borderWidth: 4,
      hoverOffset: 4,
      data: [99.2, 0.8]
    }
  ]
}

const chartOptions = {
  responsive: true,
  cutout: '75%',
  plugins: {
    legend: {
      position: 'bottom' as const,
      labels: { color: '#94A3B8', padding: 20, usePointStyle: true }
    },
    tooltip: {
      backgroundColor: '#0F172A',
      titleColor: '#F8FAFC',
      bodyColor: '#CBD5E1',
      borderColor: '#1E293B',
      borderWidth: 1,
      padding: 12
    }
  }
}

// GSAP Initialization
onMounted(() => {
  const tl = gsap.timeline({ defaults: { ease: 'power3.out' } })

  tl.to(headerRef.value, { opacity: 1, y: 0, duration: 0.8 })
    .to('.kpi-card', { opacity: 1, y: 0, duration: 0.6, stagger: 0.1 }, '-=0.5')
    .to(chartContainerRef.value, { opacity: 1, y: 0, duration: 0.6 }, '-=0.4')
    .to(tableHeaderRef.value, { opacity: 1, duration: 0.4 }, '-=0.2')
    .to('.table-row', { opacity: 1, duration: 0.4, stagger: 0.08 }, '-=0.2')
})
</script>
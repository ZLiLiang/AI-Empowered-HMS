<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, reactive, ref, watch } from 'vue'
import * as echarts from 'echarts'

const radarRef = ref(null)
let radarChart = null

const radarDimensions = ref([
  'Technical Proficiency',
  'Learning Agility',
  'Experience Reliability',
  'Contextual Comprehension',
  'Project Complexity'
])

const radarComparison = reactive({
  jd: [86, 78, 74, 65, 90],
  cv: [70, 80, 65, 72, 76]
})

const filePreview = reactive({
  type: 'pdf',
  name: 'candidate-resume.pdf',
  url: '/resume-demo.pdf',
  text: ''
})
const isResumeExpanded = ref(false)

const messages = ref([
  {
    role: 'AI Advisor',
    text: 'I see you led a major system restructure. Could you specify the underlying tech stack you used?',
    time: '09:30'
  },
  {
    role: 'Candidate',
    text: 'We migrated from a monolithic architecture to .NET 8 microservices, using SQL Server as the data foundation.',
    time: '09:32'
  }
])

const experiences = ref([
  { id: 1, title: 'Work Experience 1', start: '2019-01', end: '2021-12', type: 'Full-Time' },
  { id: 2, title: 'Work Experience 2', start: '2021-03', end: '2024-08', type: 'Full-Time' },
  { id: 3, title: 'Work Experience 3', start: '2024-09', end: 'Present', type: 'Consulting' }
])

function parseMonth(value) {
  if (value === 'Present') return new Date()
  const [year, month] = value.split('-').map(Number)
  return new Date(year, month - 1, 1)
}

function addMonths(date, offset) {
  return new Date(date.getFullYear(), date.getMonth() + offset, 1)
}

const timelineBoundary = computed(() => {
  const points = experiences.value.flatMap((item) => [parseMonth(item.start), parseMonth(item.end)])
  const min = new Date(Math.min(...points.map((d) => d.getTime())))
  const max = new Date(Math.max(...points.map((d) => d.getTime())))
  return { min, max }
})

const totalDuration = computed(() => {
  const { min, max } = timelineBoundary.value
  return Math.max(max.getTime() - min.getTime(), 1)
})

function dateToPercent(dateValue) {
  const { min } = timelineBoundary.value
  return ((dateValue.getTime() - min.getTime()) / totalDuration.value) * 100
}

function toPercent(dateValue) {
  return dateToPercent(parseMonth(dateValue))
}

const overlapSegments = computed(() => {
  const rows = [...experiences.value].map((item) => ({
    ...item,
    startDate: parseMonth(item.start),
    endDate: parseMonth(item.end)
  }))
  const segments = []

  for (let i = 0; i < rows.length; i += 1) {
    for (let j = i + 1; j < rows.length; j += 1) {
      const start = new Date(Math.max(rows[i].startDate.getTime(), rows[j].startDate.getTime()))
      const end = new Date(Math.min(rows[i].endDate.getTime(), rows[j].endDate.getTime()))
      if (start <= end) {
        segments.push({
          pair: `${rows[i].title} / ${rows[j].title}`,
          start,
          end,
          left: dateToPercent(start),
          width: dateToPercent(end) - dateToPercent(start)
        })
      }
    }
  }

  return segments
})

const monthTicks = computed(() => {
  const { min, max } = timelineBoundary.value
  const ticks = []
  let cursor = new Date(min.getFullYear(), min.getMonth(), 1)
  const end = new Date(max.getFullYear(), max.getMonth(), 1)

  while (cursor <= end) {
    ticks.push({
      label: formatMonth(cursor),
      left: dateToPercent(cursor),
      isYearStart: cursor.getMonth() === 0
    })
    cursor = addMonths(cursor, 1)
  }

  return ticks
})

const displayMonthTicks = computed(() =>
  monthTicks.value.filter((tick, index) => tick.isYearStart || index % 2 === 0)
)

const timelineMinWidth = computed(() => Math.max(920, monthTicks.value.length * 74))

const overlapBoundaries = computed(() => {
  const boundaries = overlapSegments.value.flatMap((segment) => [segment.left, segment.left + segment.width])
  const unique = [...new Set(boundaries.map((value) => value.toFixed(2)))]
  return unique.map((value) => Number(value))
})

function formatMonth(date) {
  const y = date.getFullYear()
  const m = `${date.getMonth() + 1}`.padStart(2, '0')
  return `${y}-${m}`
}

function setupRadarChart() {
  if (!radarRef.value) return
  radarChart = echarts.init(radarRef.value)
  radarChart.setOption({
    tooltip: {},
    radar: {
      center: ['50%', '55%'],
      radius: '72%',
      splitNumber: 5,
      axisName: {
        color: '#1a2233',
        fontSize: 14,
        fontWeight: 600
      },
      splitLine: { lineStyle: { color: 'rgba(40, 70, 130, 0.2)' } },
      splitArea: {
        areaStyle: { color: ['rgba(255, 255, 255, 0.1)', 'rgba(120, 150, 220, 0.06)'] }
      },
      axisLine: { lineStyle: { color: 'rgba(40, 70, 130, 0.2)' } },
      indicator: radarDimensions.value.map((name) => ({ name, max: 100 }))
    },
    legend: {
      bottom: 8,
      textStyle: {
        color: '#2a3751',
        fontWeight: 700
      }
    },
    series: [
      {
        name: 'JD vs CV',
        type: 'radar',
        data: [
          {
            value: radarComparison.jd,
            name: 'JD',
            areaStyle: {
              color: 'rgba(217, 119, 6, 0.28)'
            },
            lineStyle: { color: '#d97706', width: 3 },
            itemStyle: { color: '#f3bd77', borderColor: '#d97706', borderWidth: 2 },
            symbolSize: 8
          },
          {
            value: radarComparison.cv,
            name: 'CV',
            areaStyle: {
              color: 'rgba(37, 99, 235, 0.3)'
            },
            lineStyle: { color: '#2563eb', width: 3 },
            itemStyle: { color: '#8db3ff', borderColor: '#2563eb', borderWidth: 2 },
            symbolSize: 9
          }
        ]
      }
    ]
  })
}

function onResize() {
  radarChart?.resize()
}

watch(
  [radarDimensions, () => radarComparison.jd, () => radarComparison.cv],
  async () => {
    await nextTick()
    setupRadarChart()
  },
  { deep: true }
)

onMounted(() => {
  setupRadarChart()
  window.addEventListener('resize', onResize)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', onResize)
  radarChart?.dispose()
})

function openResumePreview() {
  isResumeExpanded.value = true
}

function closeResumePreview() {
  isResumeExpanded.value = false
}
</script>

<template>
  <div class="dashboard">
    <section class="left-group">
      <section class="panel radar-panel">
        <h2>Competency Radar</h2>
        <p class="sub">Quickly review the candidate's six-dimension capability profile</p>
        <div ref="radarRef" class="radar-canvas" />
      </section>

      <section class="panel timeline-panel">
        <h2>Work Experience Overlap Detection</h2>
        <p class="sub">Scrollable monthly X-axis with transparent overlap highlights and dashed Y-boundaries</p>

        <div class="timeline-scroll">
          <div class="timeline-grid" :style="{ minWidth: `${timelineMinWidth}px` }">
            <div class="timeline-overlap-layer">
              <div
                v-for="(segment, idx) in overlapSegments"
                :key="`overlay-${segment.pair}-${idx}`"
                class="overlap-overlay"
                :style="{ left: `${segment.left}%`, width: `${Math.max(segment.width, 1.2)}%` }"
              />
              <div
                v-for="(boundary, idx) in overlapBoundaries"
                :key="`boundary-${idx}`"
                class="overlap-boundary"
                :style="{ left: `${boundary}%` }"
              />
            </div>

            <div v-for="item in experiences" :key="item.id" class="experience-row">
              <div
                class="experience-bar"
                :style="{
                  left: `${toPercent(item.start)}%`,
                  width: `${Math.max(toPercent(item.end) - toPercent(item.start), 2)}%`
                }"
              >
                {{ item.title }}: {{ item.start }} - {{ item.end }} ({{ item.type }})
              </div>
            </div>

            <div class="overlap-title">Overlap Intervals</div>

            <div class="axis axis-monthly">
              <div
                v-for="(tick, idx) in displayMonthTicks"
                :key="`${tick.label}-${idx}`"
                class="tick month-tick"
                :class="{ 'year-start': tick.isYearStart }"
                :style="{ left: `${tick.left}%` }"
              >
                <span>{{ tick.label }}</span>
              </div>
            </div>
          </div>
        </div>

        <div class="overlap-summary">
          <h3>Detection Summary</h3>
          <p v-if="overlapSegments.length === 0">No overlapping work experience found.</p>
          <ul v-else>
            <li v-for="(segment, idx) in overlapSegments" :key="`summary-${idx}`">
              {{ idx + 1 }}. {{ segment.pair }} overlaps from {{ formatMonth(segment.start) }} to {{ formatMonth(segment.end) }}
            </li>
          </ul>
        </div>
      </section>
    </section>

    <section class="panel info-panel">
      <h2>Information Panel</h2>
      <p class="sub">Candidate resume preview and candidate-AI Q&A</p>

      <div class="info-content">
        <div class="preview-card">
          <div class="preview-toolbar">
            <strong>Resume Preview</strong>
            <button class="expand-btn" type="button" @click="openResumePreview">Expand</button>
          </div>
          <div class="preview-body">
            <template v-if="filePreview.type === 'pdf'">
              <iframe :src="filePreview.url" title="Resume Preview" />
            </template>
            <template v-else>
              <div class="placeholder">Resume is currently unavailable for preview.</div>
            </template>
          </div>
        </div>

        <div class="chat-card">
          <h3>Candidate and AI Q&A</h3>
          <div class="chat-list">
            <article v-for="(message, index) in messages" :key="`${message.time}-${index}`" class="chat-item">
              <header>
                <strong>{{ message.role }}</strong>
                <span>{{ message.time }}</span>
              </header>
              <p>{{ message.text }}</p>
            </article>
          </div>
        </div>
      </div>
    </section>

    <div v-if="isResumeExpanded" class="resume-modal" @click.self="closeResumePreview">
      <div class="resume-modal-panel">
        <header>
          <h3>Expanded Resume Preview</h3>
          <button type="button" class="close-btn" @click="closeResumePreview">Close</button>
        </header>
        <iframe :src="filePreview.url" title="Expanded Resume Preview" />
      </div>
    </div>
  </div>
</template>

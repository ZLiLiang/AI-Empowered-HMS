<script setup>
import { computed, ref } from 'vue'
import jdText from '../assets/java-engineer-jd.txt?raw'

const candidates = [
  {
    id: 'A',
    name: 'Alice Chen',
    role: 'Senior Java Engineer',
    match: 95,
    growth: [68, 79, 90]
  },
  {
    id: 'B',
    name: 'Brian Wang',
    role: 'Backend Engineer',
    match: 87,
    growth: [62, 72, 81]
  },
  {
    id: 'C',
    name: 'Cathy Liu',
    role: 'Platform Developer',
    match: 76,
    growth: [58, 65, 71]
  }
]

const sortedCandidates = computed(() => [...candidates].sort((a, b) => b.match - a.match))
const activeCandidateId = ref(sortedCandidates.value[0].id)

const jdLines = computed(() =>
  jdText
    .split('\n')
    .map((line) => line.trim())
    .filter(Boolean)
)

const activeCandidate = computed(
  () => sortedCandidates.value.find((candidate) => candidate.id === activeCandidateId.value) ?? sortedCandidates.value[0]
)

const chartX = [110, 330, 550]

const growthStages = ['CV1', 'CV2', 'CV3']

const growthDelta = computed(() => activeCandidate.value.growth[2] - activeCandidate.value.growth[0])

const growthRate = computed(() => Math.round((growthDelta.value / activeCandidate.value.growth[0]) * 100))

const stageDiffs = computed(() => [
  activeCandidate.value.growth[1] - activeCandidate.value.growth[0],
  activeCandidate.value.growth[2] - activeCandidate.value.growth[1]
])

const growthStory = computed(
  () =>
    `${activeCandidate.value.name} shows continuous growth from CV1 to CV3, with a total gain of ${growthDelta.value} points (${growthRate.value}%).`
)

const pointObjects = computed(() =>
  activeCandidate.value.growth.map((value, index) => ({
    x: chartX[index],
    y: 360 - value * 2.8,
    value
  }))
)

const linePoints = computed(() =>
  pointObjects.value.map((point) => `${point.x},${point.y}`).join(' ')
)

const areaPoints = computed(() => `110,360 ${linePoints.value} 550,360`)

const smoothPath = computed(() => {
  const points = pointObjects.value
  if (points.length < 2) return ''
  let path = `M ${points[0].x} ${points[0].y}`
  for (let index = 1; index < points.length; index += 1) {
    const previous = points[index - 1]
    const current = points[index]
    const controlX = (previous.x + current.x) / 2
    path += ` Q ${controlX} ${previous.y}, ${current.x} ${current.y}`
  }
  return path
})
</script>

<template>
  <section class="retrieve-page">
    <article class="panel retrieve-left rdb-left">
      <h2>Retrieve CV from Database</h2>
      <p class="sub">JD drives retrieval, and candidates are ranked by matching score</p>

      <div class="rdb-left-layout">
        <article class="rdb-jd-card">
          <header>
            <h3>JD File</h3>
            <span>java-engineer-jd.txt</span>
          </header>
          <ul>
            <li v-for="line in jdLines" :key="line">{{ line }}</li>
          </ul>
        </article>

        <article class="rdb-candidate-card">
          <header>
            <h3>Top Candidates</h3>
            <span>Sorted: High to Low Match</span>
          </header>

          <div class="rdb-candidate-list">
            <button
              v-for="(candidate, index) in sortedCandidates"
              :key="candidate.id"
              type="button"
              class="rdb-candidate-item"
              :class="{ active: activeCandidateId === candidate.id }"
              @click="activeCandidateId = candidate.id"
            >
              <div class="rdb-rank">{{ index + 1 }}</div>
              <div class="rdb-meta">
                <strong>{{ candidate.name }}</strong>
                <span>{{ candidate.role }}</span>
              </div>
              <div class="rdb-badge">{{ candidate.match }}% Match</div>
            </button>
          </div>
        </article>
      </div>
    </article>

    <article class="panel retrieve-right rdb-right">
      <h2>CV Growth Degree</h2>
      <p class="sub">Selected candidate: {{ activeCandidate.name }} ({{ activeCandidate.match }}% match)</p>

      <div class="rdb-right-layout">
        <article class="rdb-growth-story">
          <h3>Growth Summary</h3>
          <p>{{ growthStory }}</p>

          <div class="rdb-growth-metrics">
            <div>
              <span>Start</span>
              <strong>{{ activeCandidate.growth[0] }}</strong>
            </div>
            <div>
              <span>Current</span>
              <strong>{{ activeCandidate.growth[2] }}</strong>
            </div>
            <div>
              <span>Delta</span>
              <strong>+{{ growthDelta }}</strong>
            </div>
          </div>

          <ul>
            <li>CV1 to CV2: +{{ stageDiffs[0] }} points</li>
            <li>CV2 to CV3: +{{ stageDiffs[1] }} points</li>
            <li>Total improvement rate: {{ growthRate }}%</li>
          </ul>
        </article>

        <article class="rdb-growth-chart">
          <h3>Growth Curve</h3>
          <div class="rdb-trend-wrap">
            <svg viewBox="0 0 680 440" class="rdb-trend-svg" aria-label="Candidate CV growth chart">
              <defs>
                <linearGradient id="rdbAreaGradient" x1="0" y1="0" x2="0" y2="1">
                  <stop offset="0%" stop-color="#4f8ef4" stop-opacity="0.34" />
                  <stop offset="100%" stop-color="#4f8ef4" stop-opacity="0.05" />
                </linearGradient>
                <linearGradient id="rdbLineGradient" x1="0" y1="0" x2="1" y2="0">
                  <stop offset="0%" stop-color="#2f67db" />
                  <stop offset="100%" stop-color="#57a4ff" />
                </linearGradient>
                <filter id="rdbSoftGlow">
                  <feGaussianBlur stdDeviation="3" result="blur" />
                  <feMerge>
                    <feMergeNode in="blur" />
                    <feMergeNode in="SourceGraphic" />
                  </feMerge>
                </filter>
              </defs>

              <line x1="80" y1="360" x2="620" y2="360" class="axis-line" />
              <line x1="80" y1="70" x2="80" y2="360" class="axis-line" />

              <line x1="110" y1="360" x2="110" y2="90" class="grid-line" />
              <line x1="330" y1="360" x2="330" y2="90" class="grid-line" />
              <line x1="550" y1="360" x2="550" y2="90" class="grid-line" />
              <line x1="80" y1="290" x2="620" y2="290" class="grid-line" />
              <line x1="80" y1="220" x2="620" y2="220" class="grid-line" />
              <line x1="80" y1="150" x2="620" y2="150" class="grid-line" />

              <polygon :points="areaPoints" class="trend-area" />
              <path :d="smoothPath" class="trend-line" />

              <g v-for="(point, index) in pointObjects" :key="index">
                <circle :cx="point.x" :cy="point.y" r="11" class="trend-point-halo" />
                <circle :cx="point.x" :cy="point.y" r="7" class="trend-point" />
                <text :x="point.x" :y="point.y - 16" text-anchor="middle" class="rdb-value-text">{{ point.value }}</text>
                <text :x="point.x" y="402" text-anchor="middle" class="tick-text">{{ growthStages[index] }}</text>
              </g>

              <text x="48" y="364" text-anchor="middle" class="rdb-axis-label">0</text>
              <text x="48" y="294" text-anchor="middle" class="rdb-axis-label">25</text>
              <text x="48" y="224" text-anchor="middle" class="rdb-axis-label">50</text>
              <text x="48" y="154" text-anchor="middle" class="rdb-axis-label">75</text>
              <text x="48" y="84" text-anchor="middle" class="rdb-axis-label">100</text>
            </svg>
          </div>
        </article>
      </div>
    </article>
  </section>
</template>

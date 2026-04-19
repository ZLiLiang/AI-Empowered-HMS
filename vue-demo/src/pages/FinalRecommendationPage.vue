<script setup>
import { computed, ref } from 'vue'

const screeningScores = [
  { metric: 'Core Skill Match', score: 91, weight: 0.45 },
  { metric: 'Domain Relevance', score: 84, weight: 0.35 },
  { metric: 'Delivery Track Record', score: 88, weight: 0.2 }
]

const riskFlags = [
  { item: 'Tenure overlap needs clarification', severity: 'Medium', penalty: 8 },
  { item: 'One gap in leadership examples', severity: 'Low', penalty: 4 }
]

const interviewFeedback = ref(
  'Candidate communicated architecture trade-offs clearly and showed strong ownership in incident response. Needs deeper examples on stakeholder alignment for cross-team delivery.'
)

const screeningScore = computed(() =>
  Math.round(screeningScores.reduce((sum, row) => sum + row.score * row.weight, 0))
)

const riskPenalty = computed(() => riskFlags.reduce((sum, row) => sum + row.penalty, 0))

const feedbackScore = computed(() => {
  const text = interviewFeedback.value.toLowerCase()
  const positive = ['clear', 'strong', 'ownership', 'good', 'solid', 'excellent'].filter((w) => text.includes(w)).length
  const caution = ['needs', 'gap', 'risk', 'concern', 'weak'].filter((w) => text.includes(w)).length
  const base = 72
  return Math.max(45, Math.min(95, base + positive * 5 - caution * 4))
})

const finalScore = computed(() => {
  const score = screeningScore.value * 0.55 + feedbackScore.value * 0.3 + (100 - riskPenalty.value) * 0.15
  return Math.round(score)
})

const recommendationLabel = computed(() => {
  if (finalScore.value >= 85) return 'Strong Recommend'
  if (finalScore.value >= 75) return 'Recommend with Conditions'
  return 'Hold for Further Validation'
})

const summaryTable = computed(() => [
  {
    source: 'Screening Scores',
    score: screeningScore.value,
    insight: 'Skill coverage and domain fit are above baseline requirements.'
  },
  {
    source: 'Risk Flags',
    score: 100 - riskPenalty.value,
    insight: 'Risk exists but is manageable through focused validation questions.'
  },
  {
    source: 'Interview Feedback',
    score: feedbackScore.value,
    insight: 'Communication and ownership are good; alignment depth needs probing.'
  }
])

const barSeries = computed(() => [
  { label: 'Screening', value: screeningScore.value, tone: 'cool' },
  { label: 'Risk-Adjusted', value: 100 - riskPenalty.value, tone: 'warm' },
  { label: 'Interview', value: feedbackScore.value, tone: 'cool' }
])

const trendSeries = computed(() => [
  Math.max(58, screeningScore.value - 18),
  Math.max(64, screeningScore.value - 10),
  Math.max(70, feedbackScore.value - 4),
  finalScore.value
])

const trendLabels = ['Baseline', 'Profile Fit', 'Interview Signal', 'Final Ability']

const linePoints = computed(() => {
  const values = trendSeries.value
  const x = [30, 120, 210, 300]
  return values
    .map((value, idx) => {
      const y = 134 - Math.round((value / 100) * 102)
      return `${x[idx]},${y}`
    })
    .join(' ')
})
</script>

<template>
  <section class="panel frx-page">
    <h2>Final Recommendation</h2>
    <p class="sub">System summaries plus interview input are consolidated into one executive conclusion</p>

    <div class="frx-layout">
      <div class="frx-left">
        <article class="frx-input-card frx-screening">
          <div class="frx-tag">System</div>
          <h3>Screening Scores</h3>
          <ul>
            <li v-for="row in screeningScores" :key="row.metric">{{ row.metric }}: {{ row.score }}</li>
          </ul>
        </article>

        <article class="frx-input-card frx-risk">
          <div class="frx-tag">System</div>
          <h3>Risk Flags</h3>
          <ul>
            <li v-for="flag in riskFlags" :key="flag.item">{{ flag.item }} ({{ flag.severity }})</li>
          </ul>
        </article>

        <article class="frx-input-card frx-feedback">
          <div class="frx-tag manual">Editable</div>
          <h3>Interview Feedback</h3>
          <textarea v-model="interviewFeedback" rows="5" />
        </article>
      </div>

      <article class="frx-right">
        <header class="frx-summary-head">
          <span class="frx-check">OK</span>
          <div>
            <h3>Executive Summary</h3>
            <p>{{ recommendationLabel }} - Final Score {{ finalScore }}</p>
          </div>
        </header>

        <div class="frx-table-wrap">
          <table>
            <thead>
              <tr>
                <th>Source</th>
                <th>Score</th>
                <th>Insight</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="row in summaryTable" :key="row.source">
                <td>{{ row.source }}</td>
                <td>{{ row.score }}</td>
                <td>{{ row.insight }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="frx-charts">
          <div class="frx-bar-card">
            <h4>Score Comparison</h4>
            <div class="frx-bars">
              <div v-for="bar in barSeries" :key="bar.label" class="frx-bar-item">
                <div class="frx-bar-track">
                  <div class="frx-bar-fill" :class="bar.tone" :style="{ height: `${bar.value}%` }" />
                </div>
                <strong class="frx-bar-value">{{ bar.value }}</strong>
                <span>{{ bar.label }}</span>
              </div>
            </div>
          </div>

          <div class="frx-line-card">
            <h4>Candidate Ability Trend</h4>
            <svg viewBox="0 0 330 160" class="frx-line-svg">
              <line x1="24" y1="136" x2="312" y2="136" class="axis" />
              <line x1="24" y1="34" x2="312" y2="34" class="guide" />
              <line x1="24" y1="85" x2="312" y2="85" class="guide" />
              <polyline :points="linePoints" class="line" />
              <circle v-for="(point, idx) in linePoints.split(' ')" :key="idx" :cx="point.split(',')[0]" :cy="point.split(',')[1]" r="5" class="dot" />
              <text v-for="(point, idx) in linePoints.split(' ')" :key="`value-${idx}`" :x="point.split(',')[0]" :y="Number(point.split(',')[1]) - 10" class="value-label">{{ trendSeries[idx] }}</text>
              <text v-for="(label, idx) in trendLabels" :key="label" :x="[30, 120, 210, 300][idx]" y="154" class="stage-label">{{ label }}</text>
            </svg>
          </div>
        </div>
      </article>
    </div>
  </section>
</template>

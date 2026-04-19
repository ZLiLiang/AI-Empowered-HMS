<script setup>
import { computed } from 'vue'
import jdText from '../assets/java-engineer-jd.txt?raw'

const cvFileUrl = '/resume-demo.pdf'

const jdRequirements = computed(() =>
  jdText
    .split('\n')
    .map((line) => line.trim())
    .filter(Boolean)
)

const generatedQuestions = computed(() => [
  {
    source: 'JD + CV',
    question:
      'JD asks for high-concurrency design. Walk through one system you designed in Java to handle peak traffic, including bottleneck diagnosis and tuning strategy.'
  },
  {
    source: 'JD + CV',
    question:
      'The role requires microservice reliability. Based on your incident recovery experience, describe an outage, your root-cause process, and what architectural change prevented recurrence.'
  },
  {
    source: 'JD',
    question:
      'This position expects deep SQL and data modeling skills. Explain one complex query optimization you performed and how you validated the performance gain.'
  },
  {
    source: 'CV',
    question:
      'You mention strong ownership in delivery. Share a project where you had to balance timeline pressure, code quality, and technical debt.'
  },
  {
    source: 'JD + CV',
    question:
      'JD emphasizes cross-team collaboration, while your profile notes a gap in stakeholder alignment examples. Can you provide a concrete case showing alignment across product, QA, and operations?'
  }
])
</script>

<template>
  <section class="panel ipr-page">
    <h2>Interview Preparation</h2>
    <p class="sub">Question list is generated from both JD content and CV evidence</p>

    <div class="ipr-layout">
      <div class="ipr-left">
        <article class="ipr-card ipr-jd">
          <header>
            <h3>JD File</h3>
            <span>java-engineer-jd.txt</span>
          </header>
          <ul>
            <li v-for="line in jdRequirements" :key="line">{{ line }}</li>
          </ul>
        </article>

        <article class="ipr-card ipr-cv">
          <header>
            <h3>CV File</h3>
            <span>resume-demo.pdf</span>
          </header>
          <div class="ipr-cv-preview">
            <iframe :src="cvFileUrl" title="CV Preview" />
          </div>
        </article>
      </div>

      <article class="ipr-right">
        <header class="ipr-right-head">
          <h3>Question List</h3>
          <div class="ipr-badge">Generated from JD + CV</div>
        </header>

        <p class="ipr-emphasis">Every question below is mapped to job requirements and candidate evidence.</p>

        <ol class="ipr-questions">
          <li v-for="(item, index) in generatedQuestions" :key="item.question">
            <div class="ipr-question-top">
              <span>Q{{ index + 1 }}</span>
              <em>{{ item.source }}</em>
            </div>
            <p>{{ item.question }}</p>
          </li>
        </ol>
      </article>
    </div>
  </section>
</template>

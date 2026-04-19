<script setup>
import { computed, ref } from 'vue'
import CreateCandidateProfilePage from './pages/CreateCandidateProfilePage.vue'
import RetrieveCvFromDatabasePage from './pages/RetrieveCvFromDatabasePage.vue'
import CvScreenPage from './pages/CvScreenPage.vue'
import CvInDepthReviewPage from './pages/CvInDepthReviewPage.vue'
import CvRiskDetectionPage from './pages/CvRiskDetectionPage.vue'
import InterviewPreparationPage from './pages/InterviewPreparationPage.vue'
import FinalRecommendationPage from './pages/FinalRecommendationPage.vue'

const menuItems = [
  'Retrieve CV from Database',
  'CV Screen',
  'CV In-Depth Review',
  'Create Candidate Profile',
  'CV Risk Detection',
  'Interview Preparation',
  'Final Recommendation'
]

const activeMenu = ref('Create Candidate Profile')

const pageMap = {
  'Create Candidate Profile': CreateCandidateProfilePage,
  'Retrieve CV from Database': RetrieveCvFromDatabasePage,
  'CV Screen': CvScreenPage,
  'CV In-Depth Review': CvInDepthReviewPage,
  'CV Risk Detection': CvRiskDetectionPage,
  'Interview Preparation': InterviewPreparationPage,
  'Final Recommendation': FinalRecommendationPage
}

const activePageComponent = computed(() => pageMap[activeMenu.value] || CreateCandidateProfilePage)
</script>

<template>
  <div class="app-shell">
    <aside class="side-nav">
      <h1>Candidate Workspace</h1>
      <p class="side-sub">Workflow Navigation</p>
      <nav>
        <button
          v-for="item in menuItems"
          :key="item"
          type="button"
          class="nav-item"
          :class="{ active: activeMenu === item }"
          @click="activeMenu = item"
        >
          {{ item }}
        </button>
      </nav>
    </aside>

    <main class="content-area">
      <component :is="activePageComponent" />
    </main>
  </div>
</template>

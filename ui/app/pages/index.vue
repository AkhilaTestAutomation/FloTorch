<script setup lang="ts">

definePageMeta({
  layout: false
})

useHead({
  title: "FloTorch"
})
</script>

<template>
  <div class="flex flex-col min-h-screen bg-white">
    <header class="bg-gray-800 text-white p-2">
      <div class="container mx-auto flex justify-between items-center">
        <div>
          <img src="/logo.png" alt="logo" class="w-32" />
        </div>
        <UButton icon="i-lucide-github" variant="outline" color="neutral" href="https://github.com/FissionAI/FloTorch"
          target="_blank" />
      </div>
    </header>
    <main class="flex-1 flex flex-col items-center justify-center gap-4">
      <h1 class="text-4xl font-bold">FloTorch</h1>
      <p class="text-lg">Manage and run your RAG experiments at scale with ease</p>
      <UButton label="Get Started" trailing-icon="i-lucide-arrow-right" :to="{ name: 'projects' }" size="xl" />
    </main>
    <footer class="bg-gray-800 text-white p-2 text-sm">
      <div class="container max-w-7xl mx-auto flex justify-center items-center">
        <div>
          Powered by <a href="https://flotorch.ai?utm_source=flowtorch-repo" target="_blank"
            class="text-white underline">FloTorch.ai</a>. For
          more information, contact us at <a href="mailto:info@flotorch.ai"
            class="text-white underline">info@flotorch.ai</a>.
        </div>
      </div>
    </footer>
  </div>
</template>

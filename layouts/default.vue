<template>
  <div class="min-h-screen flex flex-col bg-gray-900 text-white">
    <TerminalNavBar />

    <main class="flex-1">
      <slot />
    </main>

    <footer class="bg-gray-900 text-gray-400 py-6 border-t border-gray-800">
      <div class="container mx-auto px-6 text-center">
        <p class="font-mono text-sm">
          <span class="text-green-400">root@terminal-master:~$</span> echo "©
          {{ new Date().getFullYear() }} Terminal Master"
        </p>
        <p class="mt-2">
          Tous droits réservés. | <span class="text-green-500">v1.0.0</span>
        </p>
      </div>
    </footer>
  </div>
</template>

<script setup>
// Utilisation du composant TerminalNavBar
</script>

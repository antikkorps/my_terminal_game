<template>
  <div class="container mx-auto p-6 py-12">
    <div class="max-w-3xl mx-auto bg-gray-800 p-6 rounded-lg shadow-lg">
      <div class="terminal-header mb-6">
        <span class="text-green-400 font-mono">root@terminal-master:~$</span>
        <TypeWriter text="cat about.md" :speed="60" />
      </div>

      <div class="prose prose-invert max-w-none">
        <h1 class="text-green-500">À propos de Terminal Master</h1>

        <p>
          Terminal Master est une application interactive conçue pour aider les débutants
          et les utilisateurs intermédiaires à maîtriser l'interface en ligne de commande
          de Linux.
        </p>

        <h2 class="text-green-400">Notre mission</h2>
        <p>
          Nous croyons que la ligne de commande est un outil puissant que tous les
          professionnels de l'informatique devraient maîtriser. Notre mission est de
          rendre l'apprentissage de la ligne de commande accessible, amusant et efficace.
        </p>

        <h2 class="text-green-400">Comment ça marche</h2>
        <p>Terminal Master propose :</p>
        <ul>
          <li>Un émulateur de terminal réaliste</li>
          <li>Des tutoriels interactifs étape par étape</li>
          <li>Un mode libre pour pratiquer sans contrainte</li>
          <li>Une progression adaptée à votre rythme</li>
        </ul>

        <h2 class="text-green-400">L'équipe</h2>
        <p>
          Terminal Master a été créé par une équipe de développeurs passionnés par Linux
          et l'éducation en informatique.
        </p>

        <div class="mt-8 font-mono bg-gray-900 p-4 rounded">
          <p class="text-green-400">root@terminal-master:~$</p>
          <TypeWriter
            text="echo 'Merci de nous rejoindre dans cette aventure!'"
            :speed="60"
            :startDelay="1000"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
// Utilisation du composant TypeWriter
</script>

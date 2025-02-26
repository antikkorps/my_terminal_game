<template>
  <div class="terminal bg-black border border-gray-700 rounded-lg overflow-hidden">
    <div
      class="terminal-header bg-gray-800 p-1.5 sm:p-2 flex justify-between items-center"
    >
      <div class="flex space-x-1.5 sm:space-x-2">
        <div class="w-2.5 h-2.5 sm:w-3 sm:h-3 bg-red-500 rounded-full"></div>
        <div class="w-2.5 h-2.5 sm:w-3 sm:h-3 bg-yellow-500 rounded-full"></div>
        <div class="w-2.5 h-2.5 sm:w-3 sm:h-3 bg-green-500 rounded-full"></div>
      </div>
      <div class="text-xs sm:text-sm text-gray-400 truncate">
        {{ freeMode ? "bash" : currentDirectory }}
      </div>
    </div>

    <div
      class="terminal-body p-2 sm:p-4 font-mono text-xs sm:text-sm overflow-y-auto h-[300px] sm:h-96 touch-auto"
      ref="terminalOutput"
      @touchstart="handleTouchStart"
      @touchmove="handleTouchMove"
    >
      <!-- Message initial en mode narratif -->
      <div
        v-if="initialMessage && !initialMessageShown"
        class="mb-3 text-blue-400 whitespace-pre-wrap"
      >
        {{ initialMessage }}
      </div>

      <!-- Historique des commandes et sorties -->
      <div v-for="(line, index) in outputLines" :key="index">
        <div v-if="line.type === 'command'" class="flex flex-wrap">
          <span class="text-green-400 mr-2 whitespace-nowrap">{{ displayPrompt }}</span>
          <span class="break-all">{{ line.content }}</span>
        </div>
        <div
          v-else-if="line.type === 'output'"
          class="text-gray-300 whitespace-pre-wrap ml-0 break-all"
        >
          {{ line.content }}
        </div>
        <div
          v-else-if="line.type === 'error'"
          class="text-red-400 whitespace-pre-wrap break-all"
        >
          {{ line.content }}
        </div>
        <div
          v-else-if="line.type === 'system'"
          class="text-blue-400 whitespace-pre-wrap break-all italic"
        >
          {{ line.content }}
        </div>
        <div
          v-else-if="line.type === 'success'"
          class="text-green-500 whitespace-pre-wrap break-all font-bold"
        >
          {{ line.content }}
        </div>
      </div>

      <!-- Ligne de commande active -->
      <div class="flex flex-wrap" v-if="!missionCompleted">
        <span class="text-green-400 mr-2 whitespace-nowrap">{{ displayPrompt }}</span>
        <input
          ref="commandInput"
          v-model="currentCommand"
          @keydown.enter="executeCommand"
          @keydown.up="navigateHistory(-1)"
          @keydown.down="navigateHistory(1)"
          @keydown.tab.prevent="handleTabCompletion"
          class="flex-1 bg-transparent text-white focus:outline-none min-w-[150px] break-all"
          autocomplete="off"
          spellcheck="false"
        />
      </div>

      <!-- Message de mission complétée -->
      <div v-else class="mt-4 text-green-400 font-bold">
        Mission accomplie! Objectif atteint.
      </div>

      <!-- Boutons spéciaux pour mobile -->
      <div
        v-if="!missionCompleted"
        class="mobile-controls mt-4 grid grid-cols-4 gap-1 sm:hidden"
      >
        <button
          v-for="(cmd, index) in commonCommands"
          :key="index"
          @click="insertCommand(cmd)"
          class="bg-gray-800 text-xs text-gray-300 py-1 px-2 rounded hover:bg-gray-700 truncate"
        >
          {{ cmd }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, nextTick, onMounted, ref, watch } from "vue"

const props = defineProps({
  // Props existants
  lesson: {
    type: Object,
    default: () => ({}),
  },
  freeMode: {
    type: Boolean,
    default: false,
  },
  // Nouveaux props pour le mode aventure
  customPrompt: {
    type: String,
    default: null,
  },
  validCommands: {
    type: Array,
    default: () => null,
  },
  initialMessage: {
    type: String,
    default: "",
  },
  successCondition: {
    type: Function,
    default: null,
  },
  fileSystem: {
    type: Object,
    default: null,
  },
})

const emit = defineEmits(["lesson-completed", "mission-completed"])

// État du terminal
const currentCommand = ref("")
const outputLines = ref([])
const commandHistory = ref([])
const historyPosition = ref(-1)
const currentDirectory = ref("~")
const currentUser = ref("user")
const terminalOutput = ref(null)
const commandInput = ref(null)
const initialMessageShown = ref(false)
const missionCompleted = ref(false)

// Commandes courantes pour les boutons mobiles
const commonCommands = ref(["ls", "cd", "pwd", "cat", "clear", "help"])

// Prompt personnalisé pour l'affichage
const displayPrompt = computed(() => {
  return props.customPrompt || `${currentUser.value}@linux:${currentDirectory.value}$`
})

// Variables pour la détection de défilement tactile
let touchStartY = 0
let scrolling = false

// Système de fichiers simulé
const fileSystemData = ref(
  props.fileSystem || {
    "~": {
      type: "directory",
      content: {
        Documents: {
          type: "directory",
          content: {
            "notes.txt": { type: "file", content: "Mes notes pour le cours Linux" },
          },
        },
        Desktop: {
          type: "directory",
          content: {},
        },
        "hello.txt": { type: "file", content: "Hello, World!" },
      },
    },
  }
)

// Commandes exécutées dans cette session
const executedCommands = ref([])

// Se concentre sur l'input et scroll en bas du terminal
const focusAndScroll = async () => {
  await nextTick()
  commandInput.value?.focus()
  if (terminalOutput.value) {
    terminalOutput.value.scrollTop = terminalOutput.value.scrollHeight
  }
}

// Gestionnaire d'événements tactiles
const handleTouchStart = (e) => {
  touchStartY = e.touches[0].clientY
  scrolling = false
}

const handleTouchMove = (e) => {
  const touchY = e.touches[0].clientY
  const diff = touchStartY - touchY

  if (Math.abs(diff) > 5) {
    scrolling = true
  }
}

// Insérer une commande depuis les boutons
const insertCommand = (cmd) => {
  currentCommand.value = cmd + " "
  commandInput.value?.focus()
}

// Exécute la commande saisie
const executeCommand = () => {
  // Si on est en train de défiler, ne pas exécuter de commande
  if (scrolling) return

  const command = currentCommand.value.trim()
  if (!command) return

  // Ajouter la commande à l'historique
  outputLines.value.push({ type: "command", content: command })
  commandHistory.value.unshift(command)
  historyPosition.value = -1
  executedCommands.value.push(command)

  // Traiter la commande
  processCommand(command)

  // S'assurer que le message initial est considéré comme affiché
  initialMessageShown.value = true

  // Vérifier si l'objectif est atteint
  checkObjective()

  currentCommand.value = ""
  focusAndScroll()
}

// Vérifie si l'objectif de la mission est atteint
const checkObjective = () => {
  if (props.successCondition && !missionCompleted.value) {
    const success = props.successCondition(executedCommands.value, outputLines.value)
    if (success) {
      missionCompleted.value = true
      outputLines.value.push({
        type: "success",
        content: "🎉 Mission accomplie! Vous avez atteint l'objectif! 🎉",
      })

      // Émettre un événement pour informer le parent
      emit("mission-completed", true)
    }
  }
}

// Traite la commande saisie
const processCommand = (command) => {
  // Vérifier si la commande est autorisée
  const firstWord = command.split(" ")[0]
  const allowedCommands = props.validCommands ||
    props.lesson?.validCommands || [
      "ls",
      "cd",
      "pwd",
      "cat",
      "clear",
      "help",
      "mkdir",
      "touch",
      "echo",
    ]

  if (!props.freeMode && !allowedCommands.includes(firstWord)) {
    outputLines.value.push({
      type: "error",
      content: `Commande '${firstWord}' non disponible dans cette mission.`,
    })
    return
  }

  // Commandes de base
  const [cmd, ...args] = command.split(" ")

  switch (cmd) {
    case "help":
      showHelp(allowedCommands)
      break
    case "clear":
      outputLines.value = []
      break
    case "ls":
      listDirectory(args[0])
      break
    case "pwd":
      outputLines.value.push({
        type: "output",
        content: `/home/${currentUser.value}${currentDirectory.value.slice(1)}`,
      })
      break
    case "cd":
      changeDirectory(args[0] || "~")
      break
    case "cat":
      catFile(args[0])
      break
    case "mkdir":
      createDirectory(args[0])
      break
    case "touch":
      createFile(args[0])
      break
    case "echo":
      outputLines.value.push({ type: "output", content: args.join(" ") })
      break
    case "rm":
      removeFile(args)
      break
    case "cp":
      copyFile(args)
      break
    case "mv":
      moveFile(args)
      break
    case "find":
      findFiles(args)
      break
    case "grep":
      grepContent(args)
      break
    case "chmod":
      changePermissions(args)
      break
    default:
      outputLines.value.push({ type: "error", content: `Commande non reconnue: ${cmd}` })
  }
}

// Commande help
const showHelp = (allowedCommands) => {
  const allCommands = {
    help: "Affiche cette liste de commandes",
    clear: "Efface l'écran du terminal",
    ls: "Liste le contenu du dossier",
    pwd: "Affiche le chemin du dossier actuel",
    cd: "Change le dossier courant",
    cat: "Affiche le contenu d'un fichier",
    mkdir: "Crée un nouveau répertoire",
    touch: "Crée un fichier vide",
    echo: "Affiche du texte dans le terminal",
    rm: "Supprime un fichier ou dossier",
    cp: "Copie un fichier ou dossier",
    mv: "Déplace ou renomme un fichier ou dossier",
    find: "Recherche des fichiers",
    grep: "Recherche du contenu dans des fichiers",
    chmod: "Modifie les permissions d'un fichier",
  }

  outputLines.value.push({
    type: "system",
    content: "📚 Commandes disponibles dans cette mission:",
  })

  const availableCommands = []
  for (const cmd of allowedCommands) {
    if (allCommands[cmd]) {
      availableCommands.push(`${cmd} - ${allCommands[cmd]}`)
    }
  }

  outputLines.value.push({
    type: "output",
    content: availableCommands.join("\n"),
  })
}

// Commande ls
const listDirectory = (path) => {
  const targetPath = resolvePath(path || currentDirectory.value)
  const dir = getNodeAtPath(targetPath)

  if (!dir || dir.type !== "directory") {
    outputLines.value.push({
      type: "error",
      content: `ls: ${path}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  const content = Object.entries(dir.content).map(([name, node]) => {
    if (node.type === "directory") {
      return `\x1b[34m${name}/\x1b[0m` // Bleu pour les dossiers
    } else {
      return name
    }
  })

  outputLines.value.push({ type: "output", content: content.join("  ") })
}

// Commande cd
const changeDirectory = (path) => {
  const targetPath = resolvePath(path)
  const dir = getNodeAtPath(targetPath)

  if (!dir) {
    outputLines.value.push({
      type: "error",
      content: `cd: ${path}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  if (dir.type !== "directory") {
    outputLines.value.push({
      type: "error",
      content: `cd: ${path}: N'est pas un dossier`,
    })
    return
  }

  currentDirectory.value = targetPath
}

// Commande cat
const catFile = (path) => {
  if (!path) {
    outputLines.value.push({ type: "error", content: "cat: argument manquant" })
    return
  }

  const targetPath = resolvePath(path)
  const file = getNodeAtPath(targetPath)

  if (!file) {
    outputLines.value.push({
      type: "error",
      content: `cat: ${path}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  if (file.type !== "file") {
    outputLines.value.push({ type: "error", content: `cat: ${path}: Est un dossier` })
    return
  }

  outputLines.value.push({ type: "output", content: file.content })
}

// Commande mkdir
const createDirectory = (path) => {
  if (!path) {
    outputLines.value.push({ type: "error", content: "mkdir: argument manquant" })
    return
  }

  const targetPath = resolvePath(path)
  const parentPath = targetPath.split("/").slice(0, -1).join("/") || "~"
  const parentDir = getNodeAtPath(parentPath)

  if (!parentDir || parentDir.type !== "directory") {
    outputLines.value.push({
      type: "error",
      content: `mkdir: ${path}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  const dirName = targetPath.split("/").pop()
  if (parentDir.content[dirName]) {
    outputLines.value.push({
      type: "error",
      content: `mkdir: ${path}: Le fichier existe déjà`,
    })
    return
  }

  parentDir.content[dirName] = { type: "directory", content: {} }
  outputLines.value.push({ type: "output", content: `mkdir: ${path}: Dossier créé` })
}

// Commande touch
const createFile = (path) => {
  if (!path) {
    outputLines.value.push({ type: "error", content: "touch: argument manquant" })
    return
  }

  const targetPath = resolvePath(path)
  const parentPath = targetPath.split("/").slice(0, -1).join("/") || "~"
  const parentDir = getNodeAtPath(parentPath)

  if (!parentDir || parentDir.type !== "directory") {
    outputLines.value.push({
      type: "error",
      content: `touch: ${path}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  const fileName = targetPath.split("/").pop()
  if (parentDir.content[fileName]) {
    outputLines.value.push({
      type: "error",
      content: `touch: ${path}: Le fichier existe déjà`,
    })
    return
  }

  parentDir.content[fileName] = { type: "file", content: "" }
  outputLines.value.push({ type: "output", content: `touch: ${path}: Fichier créé` })
}

// Commande rm
const removeFile = (args) => {
  if (args.length === 0) {
    outputLines.value.push({ type: "error", content: "rm: argument manquant" })
    return
  }

  const targetPath = resolvePath(args[0])
  const parentPath = targetPath.split("/").slice(0, -1).join("/") || "~"
  const parentDir = getNodeAtPath(parentPath)

  if (!parentDir || parentDir.type !== "directory") {
    outputLines.value.push({
      type: "error",
      content: `rm: ${args[0]}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  const fileName = targetPath.split("/").pop()
  if (!parentDir.content[fileName]) {
    outputLines.value.push({
      type: "error",
      content: `rm: ${args[0]}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  delete parentDir.content[fileName]
  outputLines.value.push({ type: "output", content: `rm: ${args[0]}: Fichier supprimé` })
}

// Commande cp
const copyFile = (args) => {
  if (args.length < 2) {
    outputLines.value.push({ type: "error", content: "cp: arguments manquants" })
    return
  }

  const sourcePath = resolvePath(args[0])
  const destPath = resolvePath(args[1])
  const sourceFile = getNodeAtPath(sourcePath)
  const destParentPath = destPath.split("/").slice(0, -1).join("/") || "~"
  const destParentDir = getNodeAtPath(destParentPath)

  if (!sourceFile) {
    outputLines.value.push({
      type: "error",
      content: `cp: ${args[0]}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  if (!destParentDir || destParentDir.type !== "directory") {
    outputLines.value.push({
      type: "error",
      content: `cp: ${args[1]}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  const destFileName = destPath.split("/").pop()
  destParentDir.content[destFileName] = { ...sourceFile }
  outputLines.value.push({ type: "output", content: `cp: ${args[0]} -> ${args[1]}` })
}

// Commande mv
const moveFile = (args) => {
  if (args.length < 2) {
    outputLines.value.push({ type: "error", content: "mv: arguments manquants" })
    return
  }

  const sourcePath = resolvePath(args[0])
  const destPath = resolvePath(args[1])
  const sourceFile = getNodeAtPath(sourcePath)
  const sourceParentPath = sourcePath.split("/").slice(0, -1).join("/") || "~"
  const sourceParentDir = getNodeAtPath(sourceParentPath)
  const destParentPath = destPath.split("/").slice(0, -1).join("/") || "~"
  const destParentDir = getNodeAtPath(destParentPath)

  if (!sourceFile) {
    outputLines.value.push({
      type: "error",
      content: `mv: ${args[0]}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  if (!destParentDir || destParentDir.type !== "directory") {
    outputLines.value.push({
      type: "error",
      content: `mv: ${args[1]}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  const destFileName = destPath.split("/").pop()
  destParentDir.content[destFileName] = { ...sourceFile }
  delete sourceParentDir.content[sourcePath.split("/").pop()]
  outputLines.value.push({ type: "output", content: `mv: ${args[0]} -> ${args[1]}` })
}

// Commande find
const findFiles = (args) => {
  if (args.length === 0) {
    outputLines.value.push({ type: "error", content: "find: argument manquant" })
    return
  }

  const searchTerm = args[0]
  const results = []

  const search = (dir, path) => {
    for (const [name, node] of Object.entries(dir.content)) {
      const fullPath = `${path}/${name}`
      if (name.includes(searchTerm)) {
        results.push(fullPath)
      }
      if (node.type === "directory") {
        search(node, fullPath)
      }
    }
  }

  search(fileSystemData.value["~"], "~")
  outputLines.value.push({ type: "output", content: results.join("\n") })
}

// Commande grep
const grepContent = (args) => {
  if (args.length < 2) {
    outputLines.value.push({ type: "error", content: "grep: arguments manquants" })
    return
  }

  const searchTerm = args[0]
  const filePath = resolvePath(args[1])
  const file = getNodeAtPath(filePath)

  if (!file || file.type !== "file") {
    outputLines.value.push({
      type: "error",
      content: `grep: ${args[1]}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  const lines = file.content.split("\n")
  const matchingLines = lines.filter((line) => line.includes(searchTerm))
  outputLines.value.push({ type: "output", content: matchingLines.join("\n") })
}

// Commande chmod
const changePermissions = (args) => {
  if (args.length < 2) {
    outputLines.value.push({ type: "error", content: "chmod: arguments manquants" })
    return
  }

  const permissions = args[0]
  const filePath = resolvePath(args[1])
  const file = getNodeAtPath(filePath)

  if (!file) {
    outputLines.value.push({
      type: "error",
      content: `chmod: ${args[1]}: Aucun fichier ou dossier de ce type`,
    })
    return
  }

  file.permissions = permissions
  outputLines.value.push({
    type: "output",
    content: `chmod: ${args[1]}: Permissions modifiées`,
  })
}

// Navigation dans l'historique des commandes
const navigateHistory = (direction) => {
  if (commandHistory.value.length === 0) return

  historyPosition.value += direction

  if (historyPosition.value < 0) {
    historyPosition.value = -1
    currentCommand.value = ""
  } else if (historyPosition.value >= commandHistory.value.length) {
    historyPosition.value = commandHistory.value.length - 1
  } else {
    currentCommand.value = commandHistory.value[historyPosition.value]
  }
}

// Autocomplétion avec Tab
const handleTabCompletion = () => {
  // Implémentation simple de l'autocomplétion
  const command = currentCommand.value.trim()
  if (!command) return

  // Autocomplétion pour les commandes
  const basicCommands = ["ls", "cd", "pwd", "cat", "echo", "clear", "help"]
  const availableCommands = props.freeMode
    ? basicCommands
    : props.lesson.validCommands || basicCommands

  // Si c'est une commande partielle
  if (!command.includes(" ")) {
    const matches = availableCommands.filter((cmd) => cmd.startsWith(command))
    if (matches.length === 1) {
      currentCommand.value = matches[0] + " "
    }
  }
  // Autocomplétion pour les chemins à implémenter
}

// Résout un chemin relatif ou absolu
const resolvePath = (path) => {
  if (!path) return currentDirectory.value

  // Chemin absolu
  if (path === "~" || path === "/home/" + currentUser.value) {
    return "~"
  }

  // Chemin relatif
  let base = currentDirectory.value
  if (path.startsWith("/")) {
    base = "~"
    path = path.replace(new RegExp(`^/home/${currentUser.value}`), "~")
  } else if (path.startsWith("~/")) {
    base = "~"
    path = path.slice(2)
  }

  // Parcourir le chemin
  const segments = path.split("/").filter(Boolean)
  for (const segment of segments) {
    if (segment === "..") {
      // Remonter d'un niveau
      base = base === "~" ? "~" : base.split("/").slice(0, -1).join("/") || "~"
    } else if (segment !== ".") {
      // Descendre d'un niveau
      base = base === "~" ? `~/${segment}` : `${base}/${segment}`
    }
  }

  return base
}

// Récupère un nœud (fichier/dossier) depuis le système de fichiers
const getNodeAtPath = (path) => {
  if (path === "~") return fileSystemData.value["~"]

  const segments = path.split("/")
  let current = fileSystemData.value["~"]

  for (let i = 1; i < segments.length; i++) {
    if (!current || current.type !== "directory") return null
    current = current.content[segments[i]]
    if (!current) return null
  }

  return current
}

// Initialisation
onMounted(() => {
  focusAndScroll()

  // Message de bienvenue spécifique à la leçon
  if (!props.freeMode && props.lesson.title) {
    outputLines.value.push({
      type: "output",
      content: `\n--- Leçon: ${props.lesson.title} ---\n`,
    })

    if (props.lesson.goals) {
      outputLines.value.push({
        type: "output",
        content:
          "Objectifs:\n" + props.lesson.goals.map((g) => `- ${g}`).join("\n") + "\n",
      })
    }
  }

  // Adapter commonCommands en fonction de la leçon
  if (!props.freeMode && props.lesson.validCommands) {
    commonCommands.value = props.lesson.validCommands.slice(0, 8) // Limiter à 8 boutons
  }
})

// Réinitialisation lors du changement de leçon
watch(
  () => props.lesson,
  () => {
    executedCommands.value = []
    if (!props.freeMode && props.lesson.title) {
      outputLines.value = [
        { type: "output", content: "Terminal Linux v1.0.0" },
        { type: "output", content: `\n--- Leçon: ${props.lesson.title} ---\n` },
      ]

      if (props.lesson.goals) {
        outputLines.value.push({
          type: "output",
          content:
            "Objectifs:\n" + props.lesson.goals.map((g) => `- ${g}`).join("\n") + "\n",
        })
      }

      outputLines.value.push({
        type: "output",
        content: 'Tapez "help" pour voir les commandes disponibles dans cette leçon.\n',
      })
    }
  },
  { deep: true }
)
</script>

<style scoped>
.terminal {
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.6);
}

/* Styles pour optimiser l'entrée sur appareils mobiles */
input,
button {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  border-radius: 0;
}

/* Optimiser le défilement tactile */
.touch-auto {
  -webkit-overflow-scrolling: touch;
}

/* Éviter que le texte ne dépasse de l'écran */
.break-all {
  word-break: break-all;
}
</style>

<template>
  <div class="terminal bg-black border border-gray-700 rounded-lg overflow-hidden">
    <div class="terminal-header bg-gray-800 p-2 flex justify-between items-center">
      <div class="flex space-x-2">
        <div class="w-3 h-3 bg-red-500 rounded-full"></div>
        <div class="w-3 h-3 bg-yellow-500 rounded-full"></div>
        <div class="w-3 h-3 bg-green-500 rounded-full"></div>
      </div>
      <div class="text-sm text-gray-400">{{ freeMode ? "bash" : currentDirectory }}</div>
    </div>

    <div
      class="terminal-body p-4 font-mono text-sm overflow-y-auto h-96"
      ref="terminalOutput"
    >
      <div v-for="(line, index) in outputLines" :key="index">
        <div v-if="line.type === 'command'" class="flex">
          <span class="text-green-400 mr-2"
            >{{ currentUser }}@linux:{{ currentDirectory }}$</span
          >
          <span>{{ line.content }}</span>
        </div>
        <div
          v-else-if="line.type === 'output'"
          class="text-gray-300 whitespace-pre-wrap ml-0"
        >
          {{ line.content }}
        </div>
        <div v-else-if="line.type === 'error'" class="text-red-400 whitespace-pre-wrap">
          {{ line.content }}
        </div>
      </div>

      <!-- Ligne de commande active -->
      <div class="flex">
        <span class="text-green-400 mr-2"
          >{{ currentUser }}@linux:{{ currentDirectory }}$</span
        >
        <input
          ref="commandInput"
          v-model="currentCommand"
          @keydown.enter="executeCommand"
          @keydown.up="navigateHistory(-1)"
          @keydown.down="navigateHistory(1)"
          @keydown.tab.prevent="handleTabCompletion"
          class="flex-1 bg-transparent text-white focus:outline-none"
          autocomplete="off"
          spellcheck="false"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { nextTick, onMounted, ref, watch } from "vue"

const props = defineProps({
  lesson: {
    type: Object,
    default: () => ({}),
  },
  freeMode: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(["lesson-completed"])

// État du terminal
const currentCommand = ref("")
const outputLines = ref([
  { type: "output", content: "Terminal Linux v1.0.0" },
  { type: "output", content: 'Tapez "help" pour afficher les commandes disponibles.\n' },
])
const commandHistory = ref([])
const historyPosition = ref(-1)
const currentDirectory = ref("~")
const currentUser = ref("user")
const terminalOutput = ref(null)
const commandInput = ref(null)

// Système de fichiers simulé
const fileSystem = ref({
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
})

// Commandes exécutées dans cette leçon
const executedCommands = ref([])

// Se concentre sur l'input et scroll en bas du terminal
const focusAndScroll = async () => {
  await nextTick()
  commandInput.value?.focus()
  if (terminalOutput.value) {
    terminalOutput.value.scrollTop = terminalOutput.value.scrollHeight
  }
}

// Exécute la commande saisie
const executeCommand = () => {
  const command = currentCommand.value.trim()
  if (!command) return

  // Ajouter la commande à l'historique
  outputLines.value.push({ type: "command", content: command })
  commandHistory.value.unshift(command)
  historyPosition.value = -1
  executedCommands.value.push(command)

  // Traiter la commande
  processCommand(command)

  // Vérifier si l'objectif de la leçon est atteint
  if (
    !props.freeMode &&
    props.lesson.successCondition &&
    props.lesson.successCondition(executedCommands.value)
  ) {
    outputLines.value.push({
      type: "output",
      content: "\n🎉 Félicitations ! Vous avez complété cette leçon ! 🎉\n",
    })
    emit("lesson-completed")
  }

  currentCommand.value = ""
  focusAndScroll()
}

// Traite la commande saisie
const processCommand = (command) => {
  // Vérifier si la commande est autorisée dans cette leçon
  if (
    !props.freeMode &&
    props.lesson.validCommands &&
    !props.lesson.validCommands.includes(command.split(" ")[0])
  ) {
    outputLines.value.push({
      type: "error",
      content: `Commande '${command}' non disponible dans cette leçon.`,
    })
    return
  }

  // Commandes de base
  const [cmd, ...args] = command.split(" ")

  switch (cmd) {
    case "help":
      showHelp()
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
    case "echo":
      outputLines.value.push({ type: "output", content: args.join(" ") })
      break
    default:
      outputLines.value.push({ type: "error", content: `Commande non reconnue: ${cmd}` })
  }
}

// Commande help
const showHelp = () => {
  let availableCommands = [
    "help - Affiche cette liste de commandes",
    "clear - Efface l'écran du terminal",
    "ls [dossier] - Liste le contenu du dossier",
    "pwd - Affiche le chemin du dossier actuel",
    "cd [dossier] - Change le dossier courant",
    "cat [fichier] - Affiche le contenu d'un fichier",
    "echo [texte] - Affiche le texte sur le terminal",
  ]

  if (!props.freeMode && props.lesson.validCommands) {
    outputLines.value.push({
      type: "output",
      content: "📚 Commandes disponibles dans cette leçon:",
    })

    availableCommands = availableCommands.filter((cmd) =>
      props.lesson.validCommands.includes(cmd.split(" ")[0])
    )
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
  if (path === "~") return fileSystem.value["~"]

  const segments = path.split("/")
  let current = fileSystem.value["~"]

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
</style>

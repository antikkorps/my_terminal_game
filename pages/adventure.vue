<template>
  <div class="bg-gray-900 text-white min-h-screen">
    <div class="container mx-auto p-3 sm:p-6">
      <h1 class="text-xl sm:text-3xl font-bold mb-4 sm:mb-6 text-green-500">
        Opération Terminal
      </h1>

      <div class="grid md:grid-cols-5 gap-4 sm:gap-6">
        <!-- Panneau latéral avec infos de mission -->
        <div class="bg-gray-800 p-3 sm:p-4 rounded-lg md:col-span-2">
          <div class="flex justify-between items-center mb-2 sm:mb-4">
            <h2 class="text-lg sm:text-xl font-bold text-yellow-400">
              Mission {{ currentMission + 1 }}
            </h2>
            <div class="bg-gray-700 px-2 py-1 rounded text-sm">
              Niveau: {{ missions[currentMission].difficulty }}
            </div>
          </div>

          <div class="prose prose-sm prose-invert mb-4">
            <h3 class="text-green-400 border-b border-gray-700 pb-1 mb-2">Contexte</h3>
            <p>{{ missions[currentMission].context }}</p>

            <h3 class="text-green-400 border-b border-gray-700 pb-1 mb-2 mt-4">
              Objectif
            </h3>
            <p>{{ missions[currentMission].objective }}</p>

            <div v-if="missions[currentMission].hints.length > 0" class="mt-4">
              <h3 class="text-green-400 border-b border-gray-700 pb-1 mb-2">Indices</h3>
              <div class="flex flex-wrap gap-2">
                <button
                  v-for="(hint, idx) in missions[currentMission].hints"
                  :key="idx"
                  @click="revealHint(idx)"
                  :class="[
                    'text-xs py-1 px-2 rounded transition-colors',
                    revealedHints.includes(idx)
                      ? 'bg-green-900 text-green-200'
                      : 'bg-gray-700 hover:bg-gray-600 text-gray-300',
                  ]"
                >
                  {{ revealedHints.includes(idx) ? hint : `Indice ${idx + 1}` }}
                </button>
              </div>
            </div>
          </div>

          <!-- Progression -->
          <div class="mt-6">
            <h3 class="text-green-400 text-sm mb-2">Progression</h3>
            <div class="w-full bg-gray-700 rounded-full h-2.5">
              <div
                class="bg-green-500 h-2.5 rounded-full"
                :style="`width: ${(currentMission / missions.length) * 100}%`"
              ></div>
            </div>
            <div class="flex justify-between text-xs mt-1 text-gray-400">
              <span>Mission {{ currentMission + 1 }}</span>
              <span>{{ missions.length }} missions</span>
            </div>
          </div>
        </div>

        <!-- Terminal et feedback -->
        <div class="md:col-span-3 space-y-4">
          <!-- Terminal -->
          <TerminalEmulator
            :customPrompt="missions[currentMission].customPrompt"
            :validCommands="missions[currentMission].validCommands"
            :initialMessage="missions[currentMission].initialMessage"
            :successCondition="missions[currentMission].successCondition"
            :fileSystem="missions[currentMission].fileSystem"
            @mission-completed="handleMissionCompleted"
          />

          <!-- Feedback section -->
          <div
            v-if="showFeedback"
            class="bg-gray-800 p-4 rounded-lg border-l-4"
            :class="{
              'border-green-500 bg-green-900/20': missionSuccess,
              'border-yellow-500 bg-yellow-900/20': !missionSuccess && !missionFailed,
              'border-red-500 bg-red-900/20': missionFailed,
            }"
          >
            <div class="flex items-start">
              <div
                class="w-8 h-8 rounded-full flex items-center justify-center mr-3 flex-shrink-0"
                :class="{
                  'bg-green-700 text-white': missionSuccess,
                  'bg-yellow-700 text-white': !missionSuccess && !missionFailed,
                  'bg-red-700 text-white': missionFailed,
                }"
              >
                <span v-if="missionSuccess">✓</span>
                <span v-else-if="missionFailed">✗</span>
                <span v-else>!</span>
              </div>
              <div>
                <h3
                  class="font-bold mb-1"
                  :class="{
                    'text-green-400': missionSuccess,
                    'text-yellow-400': !missionSuccess && !missionFailed,
                    'text-red-400': missionFailed,
                  }"
                >
                  {{ feedbackTitle }}
                </h3>
                <p class="text-sm text-gray-300">{{ feedbackMessage }}</p>
              </div>
            </div>

            <!-- Navigation buttons -->
            <div class="flex justify-between mt-4">
              <button
                v-if="currentMission > 0"
                @click="previousMission"
                class="bg-gray-700 hover:bg-gray-600 text-white text-sm px-3 py-1 rounded transition-colors"
              >
                ◄ Mission précédente
              </button>
              <div v-else></div>

              <button
                v-if="missionSuccess"
                @click="nextMission"
                class="bg-green-700 hover:bg-green-600 text-white text-sm px-3 py-1 rounded transition-colors"
              >
                Mission suivante ►
              </button>
              <button
                v-else
                @click="retryMission"
                class="bg-yellow-700 hover:bg-yellow-600 text-white text-sm px-3 py-1 rounded transition-colors"
              >
                Réessayer
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue"

// État du jeu
const currentMission = ref(0)
const completedMissions = ref([])
const revealedHints = ref([])
const showFeedback = ref(false)
const missionSuccess = ref(false)
const missionFailed = ref(false)
const feedbackTitle = ref("")
const feedbackMessage = ref("")

// Liste des missions narratives
const missions = [
  {
    title: "Une entrée discrète",
    difficulty: "Débutant",
    context:
      "Vous êtes un agent secret qui vient d'infiltrer le terminal d'un système informatique suspect. Votre première tâche consiste à explorer discrètement les lieux sans laisser de traces.",
    objective:
      "Explorez le contenu du système et trouvez un fichier nommé 'secret_coordinates.txt'.",
    initialMessage:
      "Connexion établie. Terminal sécurisé activé. Bienvenue, agent. Votre mission commence maintenant. Restez discret.\n\nÀ vous de jouer, agent.",
    customPrompt: "agent@infiltration:~$",
    validCommands: ["ls", "cd", "pwd", "cat", "clear", "help"],
    hints: [
      "Utilisez 'ls' pour lister les fichiers",
      "Utilisez 'cd' pour changer de répertoire",
      "Utilisez 'cat' pour lire le contenu d'un fichier",
    ],
    fileSystem: {
      "~": {
        type: "directory",
        content: {
          Documents: {
            type: "directory",
            content: {
              "README.txt": {
                type: "file",
                content: "Bienvenue dans le système. Tous les accès sont surveillés.",
              },
              "personnel.txt": {
                type: "file",
                content:
                  "Liste du personnel:\n- Directeur: John Smith\n- Sécurité: Alex Johnson\n- Tech: Maria Garcia",
              },
            },
          },
          Downloads: {
            type: "directory",
            content: {},
          },
          secret_data: {
            type: "directory",
            content: {
              "secret_coordinates.txt": {
                type: "file",
                content:
                  "COORDONNÉES TROUVÉES: 34°03'N 118°15'O\n\nCeci semble être la localisation d'une installation secrète à Los Angeles.",
              },
            },
          },
          system_logs: {
            type: "directory",
            content: {
              "access.log": {
                type: "file",
                content:
                  "Plusieurs tentatives d'accès non autorisées détectées.\nDernière connexion: 2023-07-15 03:24:12",
              },
            },
          },
        },
      },
    },
    successCondition: (history, output) => {
      return output.some(
        (line) => line.content && line.content.includes("COORDONNÉES TROUVÉES")
      )
    },
  },
  {
    title: "Récupération de données",
    difficulty: "Intermédiaire",
    context:
      "Bravo agent! Vous avez localisé les coordonnées. Mais l'alarme s'est déclenchée et le système se verrouille progressivement. Vous devez extraire rapidement des fichiers sensibles avant que l'accès ne soit coupé.",
    objective:
      "Copiez tous les fichiers .txt du répertoire 'classified' vers votre répertoire personnel.",
    initialMessage:
      "ALERTE! INTRUSION DÉTECTÉE!\nTemps restant avant verrouillage: 5 minutes\nExtrayez rapidement les données!\n\nNouvelle structure de fichiers chargée.",
    customPrompt: "agent@danger:~$",
    validCommands: ["ls", "cd", "pwd", "cat", "cp", "find", "mkdir", "clear", "help"],
    hints: [
      "Utilisez 'mkdir' pour créer un dossier d'extraction",
      "La commande 'cp' permet de copier des fichiers",
      "La commande 'find' peut vous aider à localiser tous les fichiers .txt",
    ],
    fileSystem: {
      "~": {
        type: "directory",
        content: {
          extraction: { type: "directory", content: {} },
          classified: {
            type: "directory",
            content: {
              project_X: {
                type: "directory",
                content: {
                  "blueprint.txt": {
                    type: "file",
                    content: "Plans détaillés du projet X: arme biologique expérimentale",
                  },
                  "participants.dat": { type: "file", content: "[DONNÉES ENCODÉES]" },
                },
              },
              agents: {
                type: "directory",
                content: {
                  "undercover.txt": {
                    type: "file",
                    content: "Liste des agents infiltrés dans l'organisation",
                  },
                  "contacts.txt": {
                    type: "file",
                    content: "Réseau de contacts internationaux",
                  },
                },
              },
              "access_codes.txt": {
                type: "file",
                content: "Codes d'accès au serveur principal: 7XV3-9YZ2-RPTA-QF5H",
              },
            },
          },
        },
      },
    },
    successCondition: (history, output) => {
      return (
        history.some((cmd) => cmd.includes("cp") && cmd.includes(".txt")) &&
        history.some((cmd) => cmd.includes("find") && cmd.includes(".txt"))
      )
    },
  },
  // Plus de missions avec des histoires et des défis de difficulté croissante
  {
    title: "Désactivation du système de sécurité",
    difficulty: "Avancé",
    context:
      "Vous avez réussi à extraire les données critiques! Mais pour échapper complètement au système de sécurité, vous devez désactiver les alarmes. Le problème: un script malveillant a été installé.",
    objective:
      "Trouvez et modifiez les permissions du script malveillant pour le rendre inoffensif.",
    initialMessage:
      "Connexion sécurisée établie. Système de sécurité actif. Désactivez-le pour permettre votre extraction.\n\nAttention: Un script malveillant est en cours d'exécution.",
    customPrompt: "root@security:~$",
    validCommands: [
      "ls",
      "cd",
      "pwd",
      "cat",
      "chmod",
      "ps",
      "grep",
      "find",
      "rm",
      "clear",
      "help",
    ],
    hints: [
      "Utilisez 'ps' pour voir les processus en cours",
      "La commande 'chmod' permet de modifier les permissions",
      "Regardez dans le dossier 'system' pour trouver le script malveillant",
    ],
    fileSystem: {
      "~": {
        type: "directory",
        content: {
          system: {
            type: "directory",
            content: {
              security: {
                type: "directory",
                content: {
                  "malicious.sh": {
                    type: "file",
                    permissions: "rwxr-xr-x",
                    content:
                      "#!/bin/bash\n# MALICIOUS SCRIPT\n# Ce script envoie toutes les données système à un serveur distant\nwhile true; do\n  send_data_to_remote_server\n  sleep 10\ndone",
                  },
                  "alarm.conf": {
                    type: "file",
                    content:
                      "ALARM_ACTIVE=true\nNOTIFY_SECURITY=true\nLOCKDOWN_ON_BREACH=true",
                  },
                },
              },
              logs: {
                type: "directory",
                content: {
                  "security.log": {
                    type: "file",
                    content:
                      "23:15:42 - Tentative d'accès non autorisée\n23:16:01 - Exécution du script malicious.sh\n23:17:30 - Transmission de données vers serveur externe",
                  },
                },
              },
            },
          },
        },
      },
    },
    successCondition: (history, output) => {
      return history.some(
        (cmd) =>
          cmd.includes("chmod") && cmd.includes("malicious.sh") && cmd.includes("-x")
      )
    },
  },
  {
    title: "L'énigme du fichier caché",
    difficulty: "Intermédiaire",
    context:
      "Votre extraction se passe bien, mais vous avez découvert qu'un fichier crucial est caché quelque part dans le système. Il contient le mot de passe pour accéder au serveur principal.",
    objective:
      "Trouvez le fichier caché qui commence par '.secret' et lisez son contenu.",
    initialMessage:
      "Intelligence reçue: Un fichier caché contenant des informations cruciales se trouve dans le système.\nIl commence par '.secret'.\n\nRappel: Les fichiers commençant par un point sont cachés dans Linux.",
    customPrompt: "agent@recherche:~$",
    validCommands: [
      "ls",
      "cd",
      "pwd",
      "cat",
      "find",
      "grep",
      "ls -a",
      "ls -la",
      "clear",
      "help",
    ],
    hints: [
      "Les fichiers cachés commencent par un point (.)",
      "La commande 'ls -a' affiche les fichiers cachés",
      "Utilisez 'find' pour rechercher des fichiers par nom",
    ],
    fileSystem: {
      "~": {
        type: "directory",
        content: {
          Documents: {
            type: "directory",
            content: {
              ".secret_keys": {
                type: "file",
                content:
                  "TROUVÉ! Mot de passe du serveur principal: XK92-LPRT-721Z\nNe stockez jamais vos mots de passe en texte clair comme ceci!",
              },
              "rapport.txt": {
                type: "file",
                content: "Rapport mensuel de sécurité. Rien d'inhabituel à signaler.",
              },
            },
          },
          config: {
            type: "directory",
            content: {
              ".secret_backup": { type: "directory", content: {} },
            },
          },
        },
      },
    },
    successCondition: (history, output) => {
      return output.some(
        (line) =>
          line.content &&
          line.content.includes("TROUVÉ! Mot de passe du serveur principal")
      )
    },
  },

  {
    title: "La bombe logique",
    difficulty: "Expert",
    context:
      "Alerte! Une bombe logique a été activée dans le système. Elle est programmée pour effacer toutes les données dans 5 minutes. Vous devez la localiser et la désactiver.",
    objective:
      "Trouvez et arrêtez le processus malveillant 'bomb.sh' et supprimez le fichier correspondant.",
    initialMessage:
      "⚠️ ALERTE CRITIQUE ⚠️\nBombe logique détectée!\nLocalisation: Inconnue\nTemps restant: 00:05:00\n\nDésactivez-la avant qu'elle n'efface toutes les données!",
    customPrompt: "expert@urgence:~$",
    validCommands: [
      "ls",
      "cd",
      "pwd",
      "cat",
      "ps",
      "grep",
      "find",
      "kill",
      "rm",
      "chmod",
      "clear",
      "help",
    ],
    hints: [
      "Utilisez 'ps' pour voir les processus en cours",
      "Recherchez dans les dossiers système",
      "Examinez les fichiers de démarrage",
    ],
    fileSystem: {
      "~": {
        type: "directory",
        content: {
          system: {
            type: "directory",
            content: {
              startup: {
                type: "directory",
                content: {
                  "bomb.sh": {
                    type: "file",
                    permissions: "rwxr-xr-x",
                    content:
                      "#!/bin/bash\n# BOMBE LOGIQUE\n# Ce script va effacer toutes les données du système\nshred -u /* -f &>/dev/null\nrm -rf /* --no-preserve-root &>/dev/null",
                  },
                  "services.conf": {
                    type: "file",
                    content: "bomb.sh:active:root:startup",
                  },
                },
              },
            },
          },
        },
      },
    },
    successCondition: (history, output) => {
      return (
        history.some((cmd) => cmd.includes("chmod") && cmd.includes("bomb.sh")) &&
        history.some((cmd) => cmd.includes("rm") && cmd.includes("bomb.sh"))
      )
    },
  },
]

// Méthodes
const revealHint = (index) => {
  if (!revealedHints.value.includes(index)) {
    revealedHints.value.push(index)
  }
}

const handleMissionCompleted = (success) => {
  showFeedback.value = true
  missionSuccess.value = success
  missionFailed.value = !success

  if (success) {
    if (!completedMissions.value.includes(currentMission.value)) {
      completedMissions.value.push(currentMission.value)
    }
    feedbackTitle.value = "Mission accomplie!"
    feedbackMessage.value =
      "Excellent travail, agent! Vous avez réussi cette mission avec succès."
  } else {
    feedbackTitle.value = "Mission incomplète"
    feedbackMessage.value =
      "Vous n'avez pas encore atteint l'objectif. Analysez la situation et réessayez."
  }
}

const nextMission = () => {
  if (currentMission.value < missions.length - 1) {
    currentMission.value++
    showFeedback.value = false
    missionSuccess.value = false
    revealedHints.value = []
  } else {
    // Jeu terminé
    feedbackTitle.value = "Toutes les missions accomplies!"
    feedbackMessage.value =
      "Félicitations, agent! Vous avez complété toutes les missions avec succès."
  }
}

const previousMission = () => {
  if (currentMission.value > 0) {
    currentMission.value--
    showFeedback.value = false
    missionSuccess.value = completedMissions.value.includes(currentMission.value)
    revealedHints.value = []
  }
}

const retryMission = () => {
  showFeedback.value = false
  missionFailed.value = false
  // Réinitialiser l'état de la mission actuelle sans changer de mission
}
</script>

<style scoped>
/* Styles spécifiques à la page d'aventure */
.prose :deep(h3) {
  margin-top: 1rem;
  margin-bottom: 0.5rem;
}

.prose :deep(p) {
  margin-bottom: 0.5rem;
}
</style>

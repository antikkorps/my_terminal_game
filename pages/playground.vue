<template>
  <div class="bg-gray-900 text-white min-h-screen">
    <div class="container mx-auto p-3 sm:p-6">
      <h1 class="text-xl sm:text-3xl font-bold mb-4 sm:mb-6 text-green-500">Mode Libre - Terminal Linux</h1>
      
      <div class="grid md:grid-cols-3 gap-4 sm:gap-6">
        <!-- Panneau d'aide -->
        <div class="bg-gray-800 p-3 sm:p-4 rounded-lg md:col-span-1">
          <h2 class="text-lg sm:text-xl font-bold mb-2 sm:mb-4 text-blue-400">Terminal sans limites</h2>
          
          <div class="prose prose-sm prose-invert mb-4">
            <p>Bienvenue dans le mode libre! Ici, vous pouvez pratiquer toutes les commandes Linux sans contraintes.</p>
            
            <h3 class="text-blue-400 border-b border-gray-700 pb-1 mb-2 mt-4">Commandes populaires</h3>
            <ul class="space-y-1 text-sm">
              <li><code class="bg-gray-900 px-1 rounded">ls</code> - Liste les fichiers</li>
              <li><code class="bg-gray-900 px-1 rounded">cd</code> - Change de répertoire</li>
              <li><code class="bg-gray-900 px-1 rounded">pwd</code> - Affiche le chemin actuel</li>
              <li><code class="bg-gray-900 px-1 rounded">cat</code> - Affiche le contenu d'un fichier</li>
              <li><code class="bg-gray-900 px-1 rounded">mkdir</code> - Crée un dossier</li>
              <li><code class="bg-gray-900 px-1 rounded">rm</code> - Supprime un fichier</li>
              <li><code class="bg-gray-900 px-1 rounded">touch</code> - Crée un fichier vide</li>
            </ul>
            
            <div class="mt-4">
              <h3 class="text-blue-400 border-b border-gray-700 pb-1 mb-2">Modèles de systèmes de fichiers</h3>
              <div class="space-y-2">
                <button @click="loadTemplate('default')" class="bg-gray-700 w-full text-left px-2 py-1 rounded text-sm hover:bg-gray-600">
                  Système par défaut
                </button>
                <button @click="loadTemplate('project')" class="bg-gray-700 w-full text-left px-2 py-1 rounded text-sm hover:bg-gray-600">
                  Projet Web
                </button>
                <button @click="loadTemplate('server')" class="bg-gray-700 w-full text-left px-2 py-1 rounded text-sm hover:bg-gray-600">
                  Configuration serveur
                </button>
              </div>
            </div>
          </div>
        </div>
        
        <!-- Terminal -->
        <div class="md:col-span-2 space-y-4">
          <TerminalEmulator 
            :freeMode="true"
            :fileSystem="currentFileSystem"
            initialMessage="Bienvenue dans le mode libre du terminal! Vous pouvez explorer et utiliser toutes les commandes Linux disponibles.\n\nTapez 'help' pour voir la liste des commandes supportées."
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

// Système de fichiers actif
const currentFileSystem = ref({});

// Différents modèles de systèmes de fichiers
const templates = {
  default: {
    "~": {
      type: "directory",
      content: {
        "Documents": {
          type: "directory",
          content: {
            "notes.txt": { type: "file", content: "Mes notes pour apprendre Linux" },
            "todo.txt": { type: "file", content: "1. Apprendre les commandes de base\n2. Pratiquer la navigation\n3. Maîtriser les redirections" }
          }
        },
        "Pictures": {
          type: "directory",
          content: {
            "screenshot.png": { type: "file", content: "[IMAGE BINAIRE]" }
          }
        },
        "hello.txt": { type: "file", content: "Hello, World!" }
      }
    }
  },
  project: {
    "~": {
      type: "directory",
      content: {
        "projet-web": {
          type: "directory",
          content: {
            "index.html": { 
              type: "file", 
              content: "<!DOCTYPE html>\n<html>\n<head>\n  <title>Mon Projet</title>\n  <link rel=\"stylesheet\" href=\"css/style.css\">\n</head>\n<body>\n  <h1>Bienvenue sur mon site</h1>\n  <script src=\"js/main.js\"></script>\n</body>\n</html>" 
            },
            "css": {
              type: "directory",
              content: {
                "style.css": { 
                  type: "file", 
                  content: "body {\n  font-family: Arial, sans-serif;\n  margin: 0;\n  padding: 20px;\n}\n\nh1 {\n  color: navy;\n}" 
                }
              }
            },
            "js": {
              type: "directory",
              content: {
                "main.js": { 
                  type: "file", 
                  content: "// JavaScript principal\nconsole.log('Site chargé');" 
                }
              }
            },
            "README.md": { 
              type: "file", 
              content: "# Mon Projet Web\n\nCeci est un exemple de projet web pour pratiquer les commandes Linux." 
            }
          }
        }
      }
    }
  },
  server: {
    "~": {
      type: "directory",
      content: {
        "etc": {
          type: "directory",
          content: {
            "apache2": {
              type: "directory",
              content: {
                "apache2.conf": { 
                  type: "file", 
                  content: "# Configuration Apache\n\nServerName localhost\nDocumentRoot /var/www/html\n\n<Directory /var/www/html>\n  Options Indexes FollowSymLinks\n  AllowOverride All\n  Require all granted\n</Directory>" 
                },
                "sites-available": {
                  type: "directory",
                  content: {
                    "000-default.conf": { 
                      type: "file", 
                      content: "<VirtualHost *:80>\n  ServerAdmin webmaster@localhost\n  DocumentRoot /var/www/html\n  ErrorLog ${APACHE_LOG_DIR}/error.log\n  CustomLog ${APACHE_LOG_DIR}/access.log combined\n</VirtualHost>" 
                    }
                  }
                }
              }
            },
            "mysql": {
              type: "directory",
              content: {
                "my.cnf": { 
                  type: "file", 
                  content: "[mysqld]\nuser = mysql\npid-file = /var/run/mysqld/mysqld.pid\nsocket = /var/run/mysqld/mysqld.sock\nport = 3306\ndatadir = /var/lib/mysql" 
                }
              }
            }
          }
        },
        "var": {
          type: "directory",
          content: {
            "log": {
              type: "directory",
              content: {
                "auth.log": { 
                  type: "file", 
                  content: "May 7 10:15:26 server sshd[1234]: Failed password for invalid user admin from 192.168.1.10 port 22 ssh2\nMay 7 10:18:43 server sshd[1235]: Accepted publickey for root from 192.168.1.5 port 22 ssh2" 
                },
                "apache2": {
                  type: "directory",
                  content: {
                    "access.log": { 
                      type: "file", 
                      content: "192.168.1.15 - - [07/May/2023:10:27:30 +0200] \"GET /index.html HTTP/1.1\" 200 1234 \"-\" \"Mozilla/5.0\"" 
                    },
                    "error.log": { 
                      type: "file", 
                      content: "[Mon May 07 10:31:25.123456 2023] [php7:error] [pid 1234] [client 192.168.1.15:50120] PHP Notice: Undefined variable: user in /var/www/html/index.php on line 15" 
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
};

// Fonction pour charger un modèle de système de fichiers
const loadTemplate = (templateName) => {
  if (templates[templateName]) {
    // Utiliser une copie profonde pour éviter les problèmes de référence
    currentFileSystem.value = JSON.parse(JSON.stringify(templates[templateName]));
  }
};

// Charger le modèle par défaut au démarrage
onMounted(() => {
  loadTemplate('default');
});
</script>
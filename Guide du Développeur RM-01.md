# Guide du Développeur RM-01

**À lire avant utilisation :**

Le RM-01 est composé d’un **Module d’Inférence**, d’un **Module d’Application** et d’une **Puce de Cryptage et de Gestion** (ci-après dénommée Module de Gestion), interconnectés via une **puce de commutateur Ethernet intégrée**, formant un sous-réseau LAN interne. Lorsqu’un utilisateur connecte un hôte (par exemple, PC, smartphone, iPad) via l’interface **USB Type-C**, le RM-01 virtualise une interface Ethernet pour l’hôte grâce à la fonctionnalité USB Ethernet. L’hôte obtient alors une adresse IP et rejoint automatiquement le sous-réseau pour l’échange de données.

Après la mise sous tension de l’appareil et la connexion à l’hôte via l’interface **USB Type-C**, le système configure automatiquement le sous-réseau local. L’**hôte utilisateur** se voit attribuer une adresse IP statique `10.10.99.100`. Le **Module d’Inférence** (IP : `10.10.99.98`) et le **Module d’Application** (IP : `10.10.99.99`)—tous deux déploient des services SSH indépendants, permettant aux utilisateurs d’y accéder directement via des clients SSH standards (par exemple, OpenSSH, PuTTY). Le Module de Gestion, en revanche, nécessite un accès via un outil de port série.

---

## Comment fournir un accès Internet au RM-01 depuis l’hôte (exemple avec macOS)

Après avoir connecté l’**hôte utilisateur** via USB Type-C, le RM-01 apparaîtra dans la liste des interfaces réseau comme :  
- **`AX88179A`** (Version Développeur)  
- **`RMinte RM-01`** (Version Commerciale)

Veuillez suivre ces étapes pour configurer le partage de connexion Internet :

1. Ouvrez **Réglages Système** (System Settings)  
2. Allez dans **Réseau** (Network) → **Partage** (Sharing)  
3. Activez **Partage de connexion Internet** (Internet Sharing)  
4. Cliquez sur l’icône **“i”** à côté des paramètres de partage pour accéder à l’interface de configuration :  
   - Définissez **“Partager votre connexion depuis”** sur : **Wi-Fi**  
   - Dans **“Aux ordinateurs utilisant”**, cochez : **AX88179A** ou **RMinte RM-01** (selon le modèle de l’appareil)  
5. Cliquez sur **Terminé** (Done)

Ensuite, retournez à la page des paramètres **Réseau** (Network) et configurez manuellement l’interface réseau du RM-01 :  
- **Adresse IP** : `10.10.99.100`  
- **Masque de sous-réseau** : `255.255.255.0`  
- **Routeur** (Router) : `10.10.99.100` (c’est-à-dire l’IP de l’hôte lui-même)

> **Remarque** :  
> Cette configuration définit l’hôte comme une passerelle, fournissant un accès réseau NAT au RM-01. La passerelle par défaut et le DNS du RM-01 sont automatiquement attribués par l’hôte via le service DHCP. La configuration manuelle de l’IP garantit que l’appareil reste dans le sous-réseau `10.10.99.0/24`, cohérent avec la communication des services internes de l’appareil.

---

## À propos de la carte de stockage **CFexpress Type-B**

La carte de stockage **CFexpress Type-B** est l’un des composants essentiels du RM-01, responsable du démarrage du système, du déploiement du framework d’inférence et des fonctions clés telles que la distribution de logiciels ISV/SV et l’authentification d’autorisation.

La carte de stockage est divisée en trois partitions indépendantes, à savoir :

`rm01sys` `rm01app` `rm01models`

`rm01sys` — Partition Système  
Le système d’exploitation et l’environnement d’exécution principal du **Module d’Inférence** sont installés dans cette partition. Il est strictement interdit aux utilisateurs ou aux développeurs d’accéder, de modifier ou de supprimer le contenu de cette partition. Toute modification non autorisée peut entraîner l’échec du démarrage du Module d’Inférence ou l’inopérabilité des fonctions d’inférence, et tout dommage matériel ou logiciel qui en résulte n’est pas couvert par les services de garantie.

`rm01app` — Partition d’Application  
Cette partition est utilisée pour stocker temporairement les fichiers d’image `Docker` soumis par les utilisateurs ou les développeurs. Une fois l’image écrite dans `rm01app`, le système RM-01 la migre automatiquement vers le stockage **NVMe SSD** intégré de l’hôte et complète le déploiement conteneurisé. Ne lancez pas ou ne modifiez pas directement les fichiers d’application dans cette partition.

`rm01models` — Partition de Modèles  
Cette partition est dédiée au stockage des modèles d’intelligence artificielle à grande échelle (par exemple, LLM, modèles multimodaux, etc.) chargés par les utilisateurs ou les développeurs. Pour plus de détails sur les formats de modèles, les limitations de taille, les procédures de chargement et les exigences de compatibilité, reportez-vous à la section “À propos des Modèles” ci-dessous.

---

## À propos du Module d’Application

Adresse IP : `10.10.99.99`  
Plage de ports : `59000-59299`

Spécifications matérielles du Module d’Application :

```
Processeur : Intel Core i3-N305 (8 cœurs, 8 threads, fréquence de base 1.8 GHz, fréquence turbo max 3.8 GHz)  
Mémoire : 16 GB / 24 GB LPDDR5-4800MT/s (intégrée, non extensible)  
Stockage : 512 GB / 1 TB / 2 TB (optionnel) NVMe SSD  
```

Identifiants d’accès SSH du Module d’Application :

```
Nom d’utilisateur par défaut : rm01  
Mot de passe par défaut : rm01 (préconfiguré en usine, uniquement pour la première connexion)  
```

**Avertissement de sécurité :**  
Pour garantir la sécurité du système, utilisez immédiatement la commande `passwd` pour modifier le mot de passe par défaut après la première connexion SSH. Le mot de passe par défaut est uniquement destiné à la configuration initiale et ne doit pas être utilisé dans les environnements de production ou de déploiement.

---

## À propos du Module d’Inférence

Adresse IP : `10.10.99.98`  
Plage de ports de service : `58000–58999`

Le **Module d’Inférence** est l’unité de calcul principale du RM-01, prenant en charge diverses configurations d’inférence IA haute performance. Les utilisateurs peuvent sélectionner le modèle approprié en fonction de l’échelle du modèle et des exigences de performance. Les quatre configurations matérielles préinstallées en usine sont les suivantes :

| Mémoire | Bande passante mémoire | Puissance de calcul   | Nombre de Tensor Cores |
|---------|------------------------|:--------------------|:----------------------|
| 32 GB   | 204.8 GB/s             | 200 TOPS (INT8)     | 56                    |
| 64 GB   | 204.8 GB/s             | 275 TOPS (INT8)     | 64                    |
| 64 GB   | 273 GB/s               | 1,200 TFLOPS (FP4)  | 64                    |
| 128 GB  | 273 GB/s               | 2,070 TFLOPS (FP4)  | 96                    |

Le RM-01 est livré avec les deux frameworks d’inférence suivants préinstallés sur la carte de stockage **CFexpress Type-B**, tous deux exécutés sur le Module d’Inférence :

`vLLM` : Démarre automatiquement, écoute par défaut le port **58000**, fournit des interfaces API compatibles avec OpenAI, et prend en charge les requêtes standard telles que POST /v1/chat/completions.  
`TEI` (Text Embedding Inference) : Nécessite un démarrage manuel, utilisé pour les services d’intégration de texte.

Méthode d’accès API :  
Après avoir chargé un modèle avec succès, le service d’inférence **vLLM** peut être accédé via l’adresse suivante :  
`http://10.10.99.98:58000/v1/chat/completions`  
Il prend en charge les appels directs à l’aide de clients OpenAI standards (par exemple, openai-python, curl, Postman).

**Avertissement de sécurité :**  
Pour garantir la sécurité et la stabilité du système, le Module d’Inférence n’offre pas d’accès SSH. Les utilisateurs et les développeurs ne peuvent pas se connecter directement ou interagir avec le système d’exploitation sous-jacent de ce module. Toute tentative de contournement des politiques de sécurité ou d’accès direct au Module d’Inférence peut entraîner des anomalies système, une corruption de données ou une interruption de service, non couverts par les services de garantie.

---

## À propos des Modèles

Le RM-01 prend en charge l’inférence de divers modèles d’intelligence artificielle, y compris, mais sans s’y limiter : **Modèles de Langage à Grande Échelle (LLM)**, **Modèles Multimodaux (MLM)**, **Modèles de Vision-Langage (VLM)**, **Modèles d’Intégration de Texte (Embedding)** et **Modèles de Reclassement (Reranker)**. Tous les fichiers de modèles doivent être stockés sur la carte de stockage **CFexpress Type-B** intégrée à l’appareil, et les utilisateurs doivent utiliser un lecteur de carte **CFexpress Type-B** compatible pour charger, gérer et mettre à jour les modèles côté hôte.

Lorsque la carte de stockage **CFexpress Type-B** est connectée au RM-01, le système la monte comme un volume de données en lecture seule nommé `models` au chemin `/home/rm01/models`. Sa structure de fichiers standard est la suivante :

```bash
models/
├── auto/                  # Répertoire pour le chargement automatique des modèles (déploiement de niveau production)
│   ├── embedding/         # Modèles d’intégration (non chargés automatiquement, voir ci-dessous)
│   ├── llm/               # Modèles de langage à grande échelle (fichiers de poids stockés directement, voir ci-dessous)
│   └── reranker/          # Modèles de reclassement (non chargés automatiquement, voir ci-dessous)
└── dev/                   # Répertoire de modèles définis par le développeur (priorité plus élevée)
    ├── embedding/
    ├── llm/
    └── reranker/
```

> **Remarque** :  
> - Le répertoire `auto/` est utilisé pour le **déploiement de modèles standardisé et léger**, automatiquement reconnu par le système.  
> - Le répertoire `dev/` est utilisé pour le **contrôle fin des comportements des modèles par les développeurs**, avec une priorité plus élevée que `auto/`. Le système ignorera les modèles dans `auto/` si `dev/` est utilisé.

---

### 1. Mode Automatique (auto)

#### Utilisation :  
Placez les **fichiers de poids complets** du modèle (par exemple, `.safetensors`, `.bin`, `.pt`, `.awq`, etc.) **directement dans le répertoire `auto/llm/`**, **sans les imbriquer dans des sous-dossiers**.

> Exemple incorrect :  
> `auto/llm/Qwen3-30B-A3B-Instruct-2507-AWQ/model.safetensors`  
>  
> Exemple correct :  
> `auto/llm/model-001-of-006.safetensors`  
> `auto/llm/config.json`  
> `auto/llm/tokenizer.json`  
> `auto/llm/vocab.json`

#### Comportement du système :  
- Au démarrage de l’appareil, le système analyse le répertoire `auto/llm/` et charge automatiquement les modèles dans des formats compatibles.  
- **Le chargement automatique n’est pas pris en charge pour les modèles d’intégration ou de reclassement**, uniquement pour les LLM.  
- Après le chargement, le modèle active **les capacités d’inférence de base par défaut** et **n’active pas les fonctionnalités avancées suivantes** :  
  - Décodage spéculatif  
  - Mise en cache de préfixe  
  - Pré-remplissage par blocs  
  - La longueur maximale du contexte (`max_model_len`) est limitée au seuil de sécurité du système (généralement ≤ 8192 tokens).  
- **Optimisation des performances limitée** : Pour garantir la stabilité du système et la concurrence multitâche, les modèles en mode automatique utilisent une stratégie d’allocation de mémoire conservatrice (`gpu_memory_utilization` ≤ 0.8).

> **Remarque importante** :  
> Le mode automatique est adapté pour **vérifier rapidement la compatibilité des modèles** ou **des scénarios de déploiement standardisés**, mais il **n’est pas adapté pour une inférence haute performance en production**. Pour des performances optimales, utilisez le **mode manuel (dev)**.

---

### 2. Mode Manuel (dev) — Mode Développeur

> **Le mode manuel a une priorité plus élevée que le mode automatique**. Lorsqu’un fichier de configuration `.yaml` existe dans les répertoires `embedding`, `llm` ou `reranker` sous `dev/`, le système **ignore complètement tous les modèles dans `auto/`**.

#### Structure des fichiers de configuration :  
Les répertoires `embedding`, `llm` et `reranker` sous `dev/` doivent contenir les trois fichiers de configuration YAML suivants pour démarrer les services de modèles correspondants :

| Nom du fichier        | Utilité                              |
|-----------------------|--------------------------------------|
| `embedding_run.yaml`  | Démarrer le service de modèle d’intégration de texte |
| `llm_run.yaml`        | Démarrer le service de modèle de langage à grande échelle |
| `reranker_run.yaml`   | Démarrer le service de modèle de reclassement |

> Tous les fichiers doivent être situés dans les dossiers correspondants sous `dev/`, et **les noms de fichiers doivent correspondre exactement**, sensibles à la casse.

#### Exemple de fichier de configuration (`llm_run.yaml`) :

```yaml
model: /home/rm01/models/dev/llm/Qwen3-30B-A3B-Instruct-2507-AWQ
port: 58000
gpu_memory_utilization: 0.85
max_model_len: 24576
served_model_name: RM-01 LLM
enable_prefix_caching: true
enable_chunked_prefill: true
max_num_batched_tokens: 512
block_size: 16
tensor_parallel_size: 1
dtype: auto
```

> **Remarque** :  
> - Le chemin `model` doit être un **chemin absolu** pointant vers le répertoire des fichiers de modèle (pas un fichier compressé).  
> - Le `port` doit être dans la plage `58000–58999` et ne doit pas entrer en conflit avec d’autres services.  
> - Le `gpu_memory_utilization` est recommandé d’être défini entre 0.7–0.9 pour maximiser le débit.  
> - Tous les paramètres suivent la **documentation officielle de vLLM** `https://docs.vllm.ai/en/latest/index.html`. Assurez-vous de la compatibilité avec la version de vLLM actuellement utilisée sur la carte de stockage **CFexpress Type-B**.

#### Processus de démarrage :  
1. Copiez les fichiers de modèle (répertoire complet) dans `dev/llm/` (ou `embedding/`, `reranker/`).  
2. Créez et placez le fichier de configuration `.yaml` correspondant.  
3. Insérez la carte de stockage **CFexpress Type-B** et redémarrez le RM-01.  
4. Le système chargera automatiquement la configuration dans `dev/` et démarrera le service correspondant.  
5. Une fois le service démarré, il peut être accédé via des interfaces telles que `http://10.10.99.98:58000/v1/chat/completions`.

> **Recommandation** : Pour un premier déploiement, utilisez les options `--load-format auto` et `--dtype auto` fournies par vLLM pour s’adapter automatiquement au format du modèle.

---

### Remarques sur la sécurité et la maintenance

- **Connexion SSH directe au Module d’Inférence interdite** : Toute gestion de modèle doit être effectuée via la carte de stockage **CFexpress Type-B**.  
- **Les fichiers de modèle doivent être des poids bruts** : N’utilisez pas de fichiers compressés (.zip/.tar.gz), de paquets cryptés ou de formats non standard.  
- **Permissions des fichiers** : Tous les fichiers de modèle doivent être lisibles (`chmod 644`), et les répertoires doivent être exécutables (`chmod 755`).  
- **Contrôle de version** : Il est recommandé d’utiliser Git ou des conventions de nommage de fichiers (par exemple, `Qwen3-30B-A3B-Instruct-v1.2-20250930`) pour gérer les versions des modèles.  
- **Recommandation de sauvegarde** : Sauvegardez les répertoires `dev/` et `auto/` avant de mettre à jour les modèles pour éviter la perte de configuration.

---

### Recommandations pour le choix du mode

| Scénario                        | Mode recommandé                     |
|--------------------------------|--------------------------------------|
| Vérifier rapidement la compatibilité des modèles | Mode Automatique (auto)            |
| Inférence haute performance en production | Mode Manuel (dev) + configuration affinée |
| Déploiement parallèle de plusieurs modèles | Mode Manuel (dev) + plusieurs fichiers `.yaml` |
| Débogage de développement, validation de prototype | Mode Manuel (dev)                 |
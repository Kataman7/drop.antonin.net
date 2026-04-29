# Rapport d'Architecture Complet — Orthup Viewer

> Document généré le 29 avril 2026. Ce rapport décrit l'architecture, les patterns de conception, les responsabilités des classes et l'analyse GRASP du Viewer 3D Orthodontique.

---

## Table des matières

1. [Vue d'ensemble](#1-vue-densemble)
2. [Stack technique](#2-stack-technique)
3. [Architecture en couches (DDD-lite)](#3-architecture-en-couches-ddd-lite)
4. [Structure des dossiers](#4-structure-des-dossiers)
5. [Patterns de conception](#5-patterns-de-conception)
6. [Analyse GRASP](#6-analyse-grasp)
7. [Diagramme de classes](#7-diagramme-de-classes)
8. [Diagrammes de séquence](#8-diagrammes-de-séquence)
9. [Responsabilités détaillées](#9-responsabilités-détaillées)
10. [Flux de données](#10-flux-de-données)
11. [Historique des refactorisations](#11-historique-des-refactorisations)
12. [Points d'amélioration](#12-points-damélioration)

---

## 1. Vue d'ensemble

Le **Orthup Viewer** est une application web **React + Three.js** qui visualise des modèles 3D dentaires (gencives + dents) avec une animation temporelle représentant le traitement orthodontique.

### Fonctionnalités principales

- **Visualisation multi-viewport** : single, double, triple, comparative
- **Mode édition** : déplacement et rotation des dents via le LeftPanel
- **Vue comparative** : comparaison avant/après traitement côte à côte
- **Animation temporelle** : lecture du traitement étape par étape
- **Détection de collisions** : visualisation des contacts inter-dentaires (IPR)
- **Multi-langue** : support de 7 langues via i18n
- **Responsive** : desktop et tablette en landscape uniquement

---

## 2. Stack technique

| Couche | Technologie | Version |
|--------|-------------|---------|
| **Framework UI** | React | 18.x |
| **Rendu 3D** | Three.js | Dernière stable |
| **Physique** | Rapier (WASM) | Dernière stable |
| **State Management** | Redux | 4.x |
| **Build Tool** | Create React App | - |
| **Styling** | CSS Modules + Variables CSS | - |
| **Internationalisation** | Custom i18n (i18nViewer.js) | - |
| **Loader 3D** | GLTFLoader (Three.js) | - |

---

## 3. Architecture en couches (DDD-lite)

L'application suit une architecture **DDD-lite** (Domain-Driven Design allégé) avec des influences de **Clean Architecture** et **GRASP**.

```
┌─────────────────────────────────────────────────────────────────┐
│                      PRESENTATION LAYER                         │
│  React Components, Hooks, CSS, Event Handlers                  │
│  App.js, Header.js, Main.js, Layer/*, Components/*             │
├─────────────────────────────────────────────────────────────────┤
│                     APPLICATION LAYER                           │
│  Redux Store, Actions, Reducers, i18n, Gestures                │
│  Store.js, Reducers/, Actions/, Gestures/                      │
├─────────────────────────────────────────────────────────────────┤
│                      DOMAIN LAYER                               │
│  Entités, Value Objects, Domain Services, Factories            │
│  Tooth.js, SceneBuilder, ToothTransformManager, CollisionHandler│
├─────────────────────────────────────────────────────────────────┤
│                   INFRASTRUCTURE LAYER                          │
│  ThreeEngine, API Services, FileLoader, MaterialPool           │
│  ThreeEngine.js, GLObjectService.js, GLTFLoader.js             │
└─────────────────────────────────────────────────────────────────┘
```

### Pourquoi DDD-lite ?

L'architecture présente les caractéristiques suivantes du DDD :

- **Entités** : `Tooth` possède une identité (son ID) et un cycle de vie
- **Value Objects** : `Vector3`, `Matrix4`, `Quaternion` sont immuables
- **Repositories** : Le `Store` Redux fait office de repository pour l'état UI
- **Domain Services** : `ToothTransformManager`, `CollisionHandler`
- **Factories** : `ToothFactory`, `SceneBuilder`
- **Aggregates** : `ComparativeSceneGroup` regroupe les dents clonées

### Ce n'est PAS du MVC

- Pas de Controller classique entre Model et View
- React utilise un **unidirectional data flow** (Redux → React)
- Le "Model" est manipulé via des **actions dispatchées**, pas directement
- Les événements utilisateur passent par des **hooks** et des **handlers** dédiés

---

## 4. Structure des dossiers

```
src/
├── Ui/                              # Couche Presentation (React)
│   ├── App.js                       # Point d'entrée racine
│   ├── Header.js                    # Barre supérieure
│   ├── Main.js                      # Canvas 3D + rendering loop
│   ├── Assets/
│   │   └── icons/                   # Icônes SVG
│   ├── Components/
│   │   ├── movements/
│   │   │   ├── Movements.js         # Desktop
│   │   │   └── MovementsMobile.js   # Mobile (≤932px)
│   │   ├── measures/
│   │   │   ├── Measures.js
│   │   │   └── MeasuresMobile.js
│   │   ├── iprAtt/
│   │   │   ├── IprAtt.js
│   │   │   └── IprAttMobile.js
│   │   └── VersionHistory/
│   └── Layer/
│       ├── Tools.js                 # LeftPanel
│       ├── Information.js           # RightPanel
│       ├── Player.js                # Footer (timeline)
│       ├── components/
│       │   ├── common/              # GlassButton, GlassSelect, DropdownMenu
│       │   ├── information/         # ViewModeSelector, VersionSelectorRight...
│       │   └── movements/           # Sliders de déplacement
│       ├── hooks/
│       │   ├── useInformation.js    # Logique RightPanel
│       │   ├── useMovements.js      # Logique mouvements
│       │   └── usePlayer.js         # Logique player
│       └── utils/
│
├── Application/                     # Couche Application
│   ├── Store/
│   │   ├── Store.js                 # Redux store
│   │   ├── Reducer/
│   │   │   ├── ViewportReducer.js
│   │   │   ├── AnimationReducer.js
│   │   │   ├── ToothReducer.js
│   │   │   └── ...
│   │   └── Actions/
│   │       ├── ViewportTypes.js
│   │       ├── ToothTypes.js
│   │       └── ...
│   ├── i18n/
│   │   └── i18nViewer.js            # 7 langues
│   ├── Hooks/
│   │   └── useViewerAnnouncements.js
│   ├── FileLoader/
│   │   └── GLTFLoader.js            # Loader custom
│   └── LocalStorage/
│       └── cookieHandler.js
│
├── Domain/                          # Couche Domain (métier)
│   ├── Mesh/                        # Modélisation des dents
│   │   ├── Model/
│   │   │   └── Tooth.js             # Entité dent
│   │   ├── Builder/
│   │   │   └── ToothVisualBuilder.js
│   │   ├── Calculation/
│   │   │   └── ToothAxesCalculator.js
│   │   ├── Query/
│   │   │   └── ToothQueryService.js
│   │   ├── Movement/
│   │   │   └── ToothMovementTracker.js
│   │   ├── State/
│   │   │   ├── MeshSelectionStore.js
│   │   │   ├── AnimationStore.js
│   │   │   └── ToothStateDispatchers.js
│   │   ├── Factory/
│   │   │   └── ToothFactory.js
│   │   └── Handler/
│   │       ├── MeshSelectorEvent.js      # Orchestrateur
│   │       ├── ToothSelectionHandler.js  # Double-clic
│   │       ├── ToothTransformHandler.js  # Undo/redo/reset
│   │       ├── ToothSaveHandler.js       # Sauvegarde API
│   │       ├── ModificationApplier.js    # Apply version
│   │       ├── KeyboardShortcuts.js      # Raccourcis
│   │       ├── Composer.js               # Outline
│   │       └── Grid.js                   # Grille comparative
│   │
│   ├── Scene/                       # Moteur 3D
│   │   ├── Engine/
│   │   │   └── ThreeEngine.js       # Singleton moteur 3D
│   │   ├── Manager/
│   │   │   ├── SceneManager.js      # Orchestrateur (20 lignes)
│   │   │   ├── SceneInitializer.js  # Init scène
│   │   │   ├── RenderLoop.js        # Boucle rendu
│   │   │   ├── AnimationManager.js  # Animation state
│   │   │   ├── VisibilityManager.js # Visibilité éléments
│   │   │   ├── ControlSyncManager.js# Synchro contrôles
│   │   │   ├── ReduxActionCreators.js# Actions Redux
│   │   │   ├── ToothTransformManager.js
│   │   │   ├── GumDeformationManager.js
│   │   │   ├── EditorManager.js
│   │   │   ├── SnapShotManager.js
│   │   │   └── InitializationOrchestrator.js
│   │   ├── Builder/
│   │   │   ├── SceneBuilder.js      # Construction scène
│   │   │   ├── ExtraData.js         # SUPPRIMÉ (refactorisé)
│   │   │   ├── GetReferencesModifications.js
│   │   │   ├── LabelBuilder.js
│   │   │   └── PhysicsWorldBuilder.js
│   │   ├── Handler/
│   │   │   ├── CameraHandler.js
│   │   │   ├── RenderHandler.js
│   │   │   ├── AnimationHandler.js
│   │   │   ├── LayerHandler.js
│   │   │   ├── ViewportHandler.js
│   │   │   ├── CollisionHandler.js
│   │   │   └── ComparativeViewHandler.js
│   │   ├── Provider/
│   │   │   ├── LightProvider.js
│   │   │   ├── MaterialProvider.js
│   │   │   ├── MaterialPool.js
│   │   │   └── RapierProvider.js
│   │   └── Utils/
│   │       └── MatrixUpdateBatcher.js
│   │
│   └── Panel/                       # Contrôles UI 3D
│       ├── Model/
│       │   └── OrbitControl.js
│       └── Handler/
│           └── ContainerManager.js
│
└── Infrastructure/                  # Couche Infrastructure
    └── Service/
        ├── GLObjectService.js       # API GL Object
        └── ProjectService.js        # API Projet
```

---

## 5. Patterns de conception

### 5.1 Singleton — ThreeEngine

```javascript
class ThreeEngine {
    constructor() {
        this.scene = null;
        this.renderer = null;
        this.cameras = null;
        // ...
    }
}
export const threeEngine = new ThreeEngine();
```

**Justification** : Il ne doit exister qu'une seule instance du moteur 3D (une scène, un renderer, un mixer).

**Responsabilité** : Centraliser l'accès à tous les objets Three.js (scène, caméras, renderer, mixer, groupes).

---

### 5.2 Module Pattern (ES6 avec état privé)

Chaque fichier est un module ES6 encapsulant son propre état :

```javascript
// ToothSelectionHandler.js
let selectionModeEnabled = false;  // État privé
let lastclickedTooth = null;       // État privé

export const enableSelectionMode = () => {
    selectionModeEnabled = true;
};

export const isSelectionModeEnabled = () => selectionModeEnabled;
```

**Justification** : L'état est caché, seules les fonctions exportées peuvent le manipuler.

---

### 5.3 Factory — ToothFactory & SceneBuilder

```javascript
// ToothFactory.js
export const getExtraData = (importedMeshesGroup, applyOnComparative = false) => {
    const teethData = {};
    // Parcourt le GLTF et crée des instances Tooth
    importedMeshesGroup.traverse((child) => {
        if (child.isMesh && child.name.includes('_tooth_')) {
            const tooth = new Tooth(child, coords, scene);
            teethData[toothId] = tooth;
        }
    });
    return teethData;
};
```

**Justification** : La création d'objets complexes (Tooth avec sphères, flèches, pivot) est centralisée.

---

### 5.4 Observer — Redux Store

```javascript
// Composant React s'abonne au Store
const { selectedVersion } = useSelector(state => state.ViewportReducer);

// Action dispatchée
Store.dispatch({ type: 'SET_SELECTED_VERSION', payload: versionId });
```

**Justification** : Découplage entre la source de l'événement et ses consommateurs.

---

### 5.5 Command — Redux Actions

Chaque action Redux est une commande encapsulée :

```javascript
// ReduxActionCreators.js
export const setKeyframeTo = (version) => ({
    type: 'SET_KEYFRAME_TO',
    payload: version
});

export const toogleAnimationVisible = (display) => ({
    type: 'TOOGLE_ANIMATION_VISIBLE',
    payload: display
});
```

**Justification** : Encapsulation de la requête en tant qu'objet, permettant le logging, l'undo, la sérialisation.

---

### 5.6 Facade — SceneManager

Avant refactor (359 lignes) : complexité cachée derrière une interface simple.

```javascript
// SceneManager.js (après refactor - 20 lignes)
export const SceneManager = async (canvas, model, projectMetas) => {
    const { renderer } = await initializeScene(canvas, model, projectMetas);
    
    const update = ({ offsetWidth, offsetHeight }) => {
        setviewportMode(Store.getState().ViewportReducer.viewportMode);
        adjustStrippingScaleForViewport();
        syncOrbitControlState();
        renderer.setSize(offsetWidth, offsetHeight);
        renderAllViewports(getScene(), renderer, offsetWidth, offsetHeight);
    };
    
    return { update };
};
```

**Justification** : Le consommateur (`Main.js`) n'a pas besoin de connaître les détails d'initialisation.

---

### 5.7 Strategy — Viewport Modes

```javascript
// RenderLoop.js
const viewports = getViewports();
// ['main'] → single
// ['main', 'bottom'] → double
// ['main', 'front', 'top'] → triple
```

**Justification** : Le comportement de rendu change dynamiquement selon la stratégie (nombre de viewports).

---

### 5.8 State Machine — AnimationReducer

```
États : PAUSE → PLAY → SEEK → PAUSE
        ↑___________________|
```

```javascript
// AnimationReducer.js
const initialState = {
    keyframe: 0,
    isPlaying: false,
    loaded: false,
    autoRotate: false
};
```

**Justification** : L'animation a un comportement déterministe avec des transitions valides.

---

### 5.9 Builder — SceneBuilder

```javascript
// SceneBuilder.js
export const addSceneElements = async (model, center, scene, renderer) => {
    // 1. Charger le GLTF
    // 2. Créer les dents (ToothFactory)
    // 3. Créer les lumières
    // 4. Créer les gencives
    // 5. Initialiser l'animation
    // 6. Configurer les layers
};
```

**Justification** : Construction étape par étape d'un objet complexe (la scène 3D complète).

---

## 6. Analyse GRASP

### 6.1 Information Expert

> **Principe** : Assigner une responsabilité à la classe qui détient l'information nécessaire.

| Classe | Information détenue | Responsabilité assignée |
|--------|---------------------|------------------------|
| **Tooth** | `refObj`, `pivot`, `mouvements[]`, `jawSide` | Se transformer, connaître sa position, gérer son historique |
| **ThreeEngine** | `scene`, `cameras`, `renderer`, `mixer` | Centraliser l'accès au moteur 3D |
| **meshSelectionStore** | `clickedTooth`, `modifications` | Gérer l'état de sélection |
| **ViewportReducer** | `viewportMode`, `selectedVersion` | Gérer les changements de vue et version |
| **AnimationReducer** | `keyframe`, `isPlaying` | Gérer l'état de l'animation |

**Exemple** : `Tooth` connaît sa position monde, donc elle calcule elle-même ses transformations (Information Expert).

---

### 6.2 Creator

> **Principe** : La classe B doit créer la classe A si B contient, agrège, ou utilise étroitement A.

| Créateur | Crée | Justification |
|----------|------|---------------|
| **SceneBuilder** | `Tooth` (via ToothFactory) | Le builder agrège la scène qui contient les dents |
| **ToothFactory** | `Tooth` | Centralise la logique de création des dents |
| **ToothVisualBuilder** | Sphères, flèches, barycentre | Le builder connaît les coordonnées nécessaires |
| **ComparativeViewHandler** | Dents clonées | Le handler agrège le groupe comparatif |
| **SceneInitializer** | Renderer, Caméra, Lumières | Orchestrateur d'initialisation |

---

### 6.3 Controller

> **Principe** : Assigner la responsabilité de gérer les événements du système à un objet qui n'est pas UI ni Domain.

| Contrôleur | Événements gérés | Type |
|------------|------------------|------|
| **SceneManager** | Initialisation complète de l'application 3D | Application Controller |
| **ToothSelectionHandler** | Double-clic sur le canvas | Input Controller |
| **KeyboardShortcuts** | Touches clavier (`g`, `a`, `Delete`) | Input Controller |
| **useInformation hook** | Changement de version dans le sélecteur | UI Controller |
| **EditorManager** | Passage mode view/edit, sauvegarde | Domain Controller |
| **ContainerManager** | Redimensionnement de la fenêtre | System Controller |

---

### 6.4 Low Coupling (Faible couplage)

> **Principe** : Minimiser les dépendances entre classes.

✅ **Bien réalisé** :
- `Tooth.js` ne dépend pas de React — pure entité domaine
- `ToothTransformHandler` ne connaît pas les composants UI
- `ReduxActionCreators.js` isole les actions du reste du code
- `SceneManager` (20 lignes) délègue tout à des modules spécialisés

⚠️ **Perfectible** :
- `ThreeEngine` (342 lignes) fait trop de choses — pourrait être split en :
  - `RendererEngine` (renderer, composer)
  - `CameraEngine` (caméras, controls)
  - `AnimationEngine` (mixer, action)

---

### 6.5 High Cohesion (Forte cohésion)

> **Principe** : Garder les classes focalisées sur une seule responsabilité.

✅ **Très bien** (après refactorisations) :

| Module | Lignes | Seule responsabilité | Note |
|--------|--------|---------------------|------|
| `ToothSelectionHandler` | 187 | Sélection de dent au double-clic | ✓ |
| `ToothTransformHandler` | 120 | Transformation (undo/redo/reset) | ✓ |
| `ToothSaveHandler` | 289 | Sauvegarde API + gestion versions | ✓ |
| `VisibilityManager` | 63 | Visibilité marks/strippings/attachments | ✓ |
| `ControlSyncManager` | 53 | Synchro OrbitControls | ✓ |
| `AnimationManager` | 23 | État animation (mixer + keyframe) | ✓ |
| `ReduxActionCreators` | 31 | Création des actions Redux | ✓ |

---

### 6.6 Indirection

> **Principe** : Introduire un intermédiaire pour réduire le couplage.

- **Redux Store** : intermédiaire entre React et le domaine
- **ThreeEngine** : intermédiaire entre le domaine et Three.js
- **ToothQueryService** : intermédiaire entre les consommateurs et les getters de Tooth

---

### 6.7 Protected Variations

> **Principe** : Protéger les éléments des variations sur d'autres éléments.

- **MaterialPool** : isole la création des matériaux Three.js
- **ToothFactory** : isole la création des dents du format GLTF
- **ReduxActionCreators** : isole la structure des actions du reste du code

---

## 7. Diagramme de classes

### 7.1 Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────────┐
│                         UI LAYER (React)                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │    App.js    │  │   Header.js  │  │   Main.js (Canvas 3D)    │  │
│  │  (racine)    │  │ (top panel)  │  │   (ThreeEngine + loop)   │  │
│  └──────┬───────┘  └──────────────┘  └──────────────────────────┘  │
│         │                                                           │
│  ┌──────┴───────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │ Information  │  │    Tools     │  │         Player           │  │
│  │ (RightPanel) │  │ (LeftPanel)  │  │      (Footer)            │  │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘  │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Custom Hooks                              │   │
│  │  useInformation.js | useMovements.js | usePlayer.js         │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ dispatch / subscribe
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER (Redux)                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │ Viewport     │  │ Animation    │  │         Tooth            │  │
│  │  Reducer     │  │  Reducer     │  │        Reducer           │  │
│  │(mode,version)│  │(keyframe,    │  │   (selected tooth,       │  │
│  │              │  │   play/pause) │  │    denied services)      │  │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘  │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              ReduxActionCreators.js                          │   │
│  │  setVersions() | setKeyframeToEnd() | toogleAnimationVisible()│   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ read / write
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        DOMAIN LAYER                                 │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    ThreeEngine (Singleton)                  │   │
│  │  ─────────────────────────────────────────────────────────  │   │
│  │  - scene: Scene                                             │   │
│  │  - renderer: WebGLRenderer                                  │   │
│  │  - cameras: Map<string, Camera>                             │   │
│  │  - controls: OrbitControls | TrackballControls              │   │
│  │  - mixer: AnimationMixer                                    │   │
│  │  - action: AnimationAction                                  │   │
│  │  - mandiGroup: Group                                        │   │
│  │  - marks: Group                                             │   │
│  │  - strippings: Array<Group>                                 │   │
│  │  - allAttachments: Array<Object3D>                          │   │
│  │  - maxiExtraDataComparative: Object                         │   │
│  │  - mandiExtraDataComparative: Object                        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│         ┌────────────────────┼────────────────────┐                │
│         │                    │                    │                │
│  ┌──────▼──────┐   ┌────────▼────────┐   ┌──────▼──────┐         │
│  │   Tooth     │   │  SceneBuilder   │   │ Animation   │         │
│  │  (Entity)   │   │  (Factory)      │   │ Handler     │         │
│  │  ─────────  │   │  ─────────────  │   │  ─────────  │         │
│  │  +refObj    │   │  +loadGLTF()    │   │  +showGums()│         │
│  │  +pivot     │   │  +createLights()│   │  +getTimes()│         │
│  │  +mouvements│   │  +buildScene()  │   │  +isVisible()│        │
│  │  +pointSpheres│  │  +dispatchUserData()│            │         │
│  │  +vectors   │   │                 │   │             │         │
│  │  +jawSide   │   │                 │   │             │         │
│  │  ─────────  │   │                 │   │             │         │
│  │  +getMallocMatrix()│              │   │             │         │
│  │  +getAxesCalculator()│            │   │             │         │
│  │  +tooglePtsVisibility()│          │   │             │         │
│  │  +updateWorldPosition()│          │   │             │         │
│  │  +placePivot()│                   │   │             │         │
│  │  +offsetRefObj()│                 │   │             │         │
│  │  +hasMovedRelativeToMaloc()│      │   │             │         │
│  └─────────────┘   └─────────────────┘   └─────────────┘         │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              ToothSelectionHandler (Controller)             │   │
│  │  ─────────────────────────────────────────────────────────  │   │
│  │  +registerDoubleClickHandler()                             │   │
│  │  +enableSelectionMode()                                    │   │
│  │  +disableSelectionMode()                                   │   │
│  │  +isSelectionModeEnabled()                                 │   │
│  │  +setAnimationToEnd()                                      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              ToothTransformHandler (Service)                │   │
│  │  ─────────────────────────────────────────────────────────  │   │
│  │  +setPivotOnBarycenter()                                   │   │
│  │  +resetSelectedToothToInitialPosition()                    │   │
│  │  +restoreSelectedToothToNextPosition()                     │   │
│  │  +undoSelectedToothLastMovement()                          │   │
│  │  +resetAllTeethToInitialPosition()                         │   │
│  │  +hasAnyToothBeenModified()                                │   │
│  │  +applyOffsetToSelectedToothKeepIndices()                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              ToothSaveHandler (Repository-like)             │   │
│  │  ─────────────────────────────────────────────────────────  │   │
│  │  +saveModifications()                                      │   │
│  │  +removeModification()                                     │   │
│  │  +refreshModifications()                                   │   │
│  │  +clearSessionMovementHistory()                            │   │
│  │  +addToSessionMovementHistory()                            │   │
│  │  +collectToothDataForSave()                                │   │
│  │  +collectMalocAndSetupVersionsData()                       │   │
│  │  +setSaveHandlerData()                                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              ModificationApplier (Service)                  │   │
│  │  ─────────────────────────────────────────────────────────  │   │
│  │  +applyModification(id, applyOnComparative)                │   │
│  │  ─── Gère le reset et l'application des versions           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              CollisionHandler (Domain Service)              │   │
│  │  ─────────────────────────────────────────────────────────  │   │
│  │  +computeAndDisplayTeethIntersection()                     │   │
│  │  +computeIntersectionsWith()                               │   │
│  │  +initColliders()                                          │   │
│  │  ─── Physique Rapier, boîtes englobantes, contacts         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              KeyboardShortcuts (Controller)                 │   │
│  │  ─────────────────────────────────────────────────────────  │   │
│  │  +setupKeyboardShortcuts()                                 │   │
│  │  ─── Écoute 'keydown' pour 'g', 'a', 'Delete'              │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ HTTP / WebSocket
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      INFRASTRUCTURE LAYER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │ GLTFLoader   │  │ GLObject     │  │    ProjectService        │  │
│  │ (Three.js)   │  │ Service      │  │    (API REST)            │  │
│  │  ──────────  │  │  ──────────  │  │    ─────────────         │  │
│  │  +load()     │  │  +createGLObject()│  │  +PostReferencePhoto() │  │
│  │  +parse()    │  │  +deformGum()  │  │  +GetProject()         │  │
│  └──────────────┘  └──────────────┘  └──────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.2 Relations détaillées

```
ThreeEngine ──1:1──► Scene
ThreeEngine ──1:N──► Camera[]
ThreeEngine ──1:1──► WebGLRenderer
ThreeEngine ──1:1──► AnimationMixer
ThreeEngine ──1:1──► OrbitControls
ThreeEngine ──1:N──► Group (mandiGroup, tempGroup, marks...)

SceneBuilder ──crée──► Tooth[] (via ToothFactory)
ToothFactory ──utilise──► ToothVisualBuilder
ToothFactory ──utilise──► ToothAxesCalculator

Tooth ──1:1──► Mesh (refObj)
Tooth ──1:1──► Object3D (pivot)
Tooth ──1:N──► Mesh[] (pointSpheres)
Tooth ──1:N──► Vector3[] (vectors)

SceneManager ──utilise──► SceneInitializer
SceneManager ──utilise──► RenderLoop
SceneManager ──utilise──► AnimationManager
SceneManager ──utilise──► VisibilityManager
SceneManager ──utilise──► ControlSyncManager

useInformation ──dispatch──► Redux Store
useMovements ──lit──► Redux Store (ToothReducer)
Player ──dispatch──► Redux Store (AnimationReducer)
```

---

## 8. Diagrammes de séquence

### 8.1 Initialisation de la scène

```
Utilisateur          ContainerManager    SceneManager    SceneInitializer    SceneBuilder    ThreeEngine
    │                      │                   │                  │                 │              │
    │─── charge page ─────►│                   │                  │                 │              │
    │                      │─── initScene() ──►│                  │                 │              │
    │                      │                   │─── initializeScene() ──►│           │              │
    │                      │                   │                  │─── addSceneElements() ──►        │
    │                      │                   │                  │                 │─── load GLTF  │
    │                      │                   │                  │                 │◄── modèle ───│
    │                      │                   │                  │                 │              │
    │                      │                   │                  │◄── maxi/mandiExtraData ──        │
    │                      │                   │◄── renderer + data ──│                 │            │
    │                      │                   │                  │                 │              │
    │                      │                   │─── OnMeshClick() ──►│                 │              │
    │                      │                   │                  │                 │              │
    │                      │◄── update() fn ───│                  │                 │              │
    │                      │                   │                  │                 │              │
    │◄── scène affichée ──│                   │                  │                 │              │
```

### 8.2 Sélection d'une dent (double-clic)

```
Utilisateur          Canvas          ToothSelectionHandler    meshSelectionStore    ToothTransformHandler    Redux Store
    │                   │                       │                       │                       │              │
    │── double-clic ───►│                       │                       │                       │              │
    │                   │── dblclick event ────►│                       │                       │              │
    │                   │                       │── raycaster + find ───►│                       │              │
    │                   │                       │◄── tooth intersectée ─│                       │              │
    │                   │                       │                       │                       │              │
    │                   │                       │── setClickedTooth() ─►│                       │              │
    │                   │                       │                       │                       │              │
    │                   │                       │── buildTeethOutline() │                       │              │
    │                   │                       │                       │                       │              │
    │                   │                       │── dispatch() ──────────────────────────────────►│              │
    │                   │                       │                       │                       │── SET_SELECTED_TOOTH
    │                   │                       │                       │                       │              │
    │                   │                       │                       │◄── tooth sélectionnée  │              │
    │                   │                       │                       │                       │              │
    │◄── dent outline ──│                       │                       │                       │              │
    │◄── UI Movements ──│                       │                       │                       │              │
```

### 8.3 Déplacement d'une dent (slider)

```
Utilisateur          Movements.js    useMovements    ToothTransformManager    Tooth    ThreeEngine
    │                   │                  │                    │                │            │
    │── glisse slider ─►│                  │                    │                │            │
    │                   │── onChange() ──►│                    │                │            │
    │                   │                  │── applyTranslationOrRotation() ──►│            │
    │                   │                  │                    │── calcule nouvelle matrice   │
    │                   │                  │                    │                │            │
    │                   │                  │                    │── pivot.matrix.copy() ─────►│
    │                   │                  │                    │                │            │
    │                   │                  │                    │                │── render() │
    │                   │                  │                    │                │            │
    │◄── dent bouge ────│                  │                    │                │            │
```

### 8.4 Sauvegarde des modifications

```
Utilisateur          EditionButton    ToothSaveHandler    ProjectService    API Server
    │                   │                  │                   │                │
    │── clique Save ───►│                  │                   │                │
    │                   │── saveModifications() ──►│            │                │
    │                   │                  │── collectToothDataForSave()        │
    │                   │                  │                   │                │
    │                   │                  │── PostReferenceModifications() ──►│
    │                   │                  │                   │                │
    │                   │                  │◄── response ──────│                │
    │                   │                  │                   │                │
    │                   │◄── success ──────│                   │                │
    │◄── popup saved ───│                  │                   │                │
```

---

## 9. Responsabilités détaillées

### 9.1 Couche Presentation (React)

| Composant | Lignes | Responsabilité | État local |
|-----------|--------|----------------|------------|
| **App.js** | ~60 | Point d'entrée, gestion orientation, LandscapeAlert | Non |
| **Header.js** | ~80 | Barre supérieure, infos patient, boutons connexion | Non (Redux) |
| **Main.js** | ~40 | Canvas 3D, intégration SceneManager | renderer ref |
| **Tools.js** | ~120 | LeftPanel, boutons outils, specs | isExpanded |
| **Information.js** | ~100 | RightPanel, sélecteurs version, infos | Non (hooks) |
| **Player.js** | ~120 | Footer, timeline, play/pause, keyframe | isDragging |

### 9.2 Couche Application (Redux)

| Reducer | État | Actions | Consommateurs |
|---------|------|---------|---------------|
| **ViewportReducer** | mode, version, visibilités | SET_VIEWPORT_MODE, SET_SELECTED_VERSION | Information.js, SceneManager |
| **AnimationReducer** | keyframe, isPlaying, loaded, autoRotate | SET_KEYFRAME_TO, TOOGLE_ANIMATION_VISIBLE | Player.js, SceneManager |
| **ToothReducer** | selectedTooth, deniedServices | SET_SELECTED_TOOTH | Movements.js, ToothSelectionHandler |
| **CollisionReducer** | interference, striping | SET_INTERFERENCE, SET_STRIPING | CollisionHandler |

### 9.3 Couche Domain (Métier)

#### Entités

| Classe | Type | Responsabilité | Collaborateurs |
|--------|------|----------------|----------------|
| **Tooth** | Entity | Représenter une dent avec son pivot, mouvements, points de référence | ThreeEngine, MaterialPool, PhysicsWorldBuilder |

#### Factories

| Classe | Type | Responsabilité | Collaborateurs |
|--------|------|----------------|----------------|
| **SceneBuilder** | Factory | Construire la scène complète à partir du modèle GLTF | ToothFactory, LightingSetup, LayerHandler |
| **ToothFactory** | Factory | Créer les instances `Tooth` à partir des meshes GLTF | ToothVisualBuilder, ToothAxesCalculator, ToothQueryService |
| **ToothVisualBuilder** | Builder | Créer les sphères, flèches et barycentre d'une dent | Three.js (SphereGeometry, ArrowHelper) |
| **ToothAxesCalculator** | Service | Calculer les matrices et axes locaux d'une dent | Three.js (Matrix4, Vector3) |

#### Services

| Classe | Type | Responsabilité | Collaborateurs |
|--------|------|----------------|----------------|
| **SceneManager** | Facade / Controller | Orchestrer l'initialisation et la boucle de rendu | SceneInitializer, RenderLoop, AnimationManager, VisibilityManager, ControlSyncManager |
| **SceneInitializer** | Service | Initialiser le renderer, lighting, caméra, snapshot, comparative view | SceneBuilder, ThreeEngine, CameraHandler |
| **RenderLoop** | Service | Gérer le rendu multi-viewport | ThreeEngine, CameraHandler, Composer, Grid |
| **AnimationManager** | Service | Synchroniser l'animation Three.js avec le state Redux | ThreeEngine, AnimationHandler, VisibilityManager |
| **VisibilityManager** | Service | Mettre à jour la visibilité des marks, strippings, attachments | ThreeEngine, Store, AnimationHandler |
| **ControlSyncManager** | Service | Synchroniser les OrbitControls selon le mode viewport | ThreeEngine, Store, ViewportHandler |
| **ReduxActionCreators** | Factory | Créer les objets action Redux | - |
| **ToothSelectionHandler** | Controller | Gérer le double-clic et la sélection de dent | meshSelectionStore, ToothTransformHandler, Redux Store |
| **ToothTransformHandler** | Domain Service | Appliquer les transformations (translation, rotation, undo, reset) | Tooth, ThreeEngine |
| **ToothSaveHandler** | Repository | Sauvegarder les modifications sur le serveur | ProjectService, Store, ToothTransformHandler |
| **ToothTransformManager** | Domain Service | Calculer les nouvelles positions lors du déplacement | ToothQueryService, TransformUtils, ToothMovementTracker |
| **CollisionHandler** | Domain Service | Détecter les collisions entre dents via Rapier | PhysicsWorld, Tooth, Store |
| **GumDeformationManager** | Domain Service | Déformer les gencives quand les dents bougent | GLObjectService, ThreeEngine |
| **ModificationApplier** | Service | Appliquer/reset une version de modification | ToothTransformHandler, Redux Store, ThreeEngine |
| **ComparativeViewHandler** | Service | Cloner les dents/gencives pour la vue comparative | ThreeEngine, SceneBuilder |

#### Singletons

| Classe | Type | Responsabilité |
|--------|------|----------------|
| **ThreeEngine** | Singleton | Encapsuler le moteur 3D (scene, renderer, cameras, mixer, controls, groupes) |

### 9.4 Couche Infrastructure

| Classe | Type | Responsabilité | Collaborateurs |
|--------|------|----------------|----------------|
| **GLTFLoader.js** | Adapter | Charger et parser les fichiers GLTF | Three.js |
| **GLObjectService.js** | Service API | Communiquer avec le service GL Object (déformation, offset) | Fetch API |
| **ProjectService.js** | Service API | Communiquer avec l'API projet (sauvegarde, photos) | Fetch API |
| **RapierProvider.js** | Provider | Fournir le moteur physique Rapier | WASM |
| **MaterialPool.js** | Pool | Gérer le recyclage des matériaux Three.js | Three.js |

---

## 10. Flux de données

### 10.1 Architecture unidirectionnelle (Redux)

```
Action ──► Dispatcher ──► Store ──► View
  ↑                                    │
  └─────────── User Event ─────────────┘
```

### 10.2 Flux de rendu 3D

```
Main.js ──► requestAnimationFrame ──► SceneManager.update()
                                          │
                                          ▼
                                    RenderLoop.renderAllViewports()
                                          │
                                          ▼
                                    ┌─────┴─────┐
                                    │           │
                                    ▼           ▼
                              setupViewport()  setupCamera()
                                    │           │
                                    └─────┬─────┘
                                          ▼
                                    updateAnimationState()
                                          │
                                          ▼
                                    ┌─────┴─────┐
                                    │           │
                                    ▼           ▼
                              showGengivas()  showPontics()
                                    │
                                    ▼
                              updateElementsVisibility()
                                    │
                                    ▼
                              composerRender()
```

### 10.3 Flux de sélection de dent

```
Double-clic ──► ToothSelectionHandler ──► meshSelectionStore.setClickedTooth()
                                                │
                                                ▼
                                          Redux Store.dispatch()
                                                │
                                                ▼
                                          ToothReducer ──► state.selectedTooth
                                                │
                                                ▼
                                          useMovements (hook)
                                                │
                                                ▼
                                          Movements.js (affichage sliders)
```

---

## 11. Historique des refactorisations

### 11.1 Phase 1 : MeshSelectorEvent.js (1139 → 5 modules)

| Fichier créé | Lignes | Responsabilité |
|--------------|--------|----------------|
| `ToothSelectionHandler.js` | 187 | Raycaster, double-clic, sélection/désélection |
| `ToothTransformHandler.js` | 120 | TransformControls, drag handling, undo/redo/reset |
| `ToothSaveHandler.js` | 289 | Collecte de données, API serveur, dropdown versions |
| `ModificationApplier.js` | 227 | `applyModification`, reset original, dispatch versions |
| `KeyboardShortcuts.js` | 45 | Raccourcis clavier isolés |
| **MeshSelectorEvent.js (nouveau)** | **20** | **Orchestrateur** |

**Économie** : 1139 → ~698 lignes (~38% de réduction)

### 11.2 Phase 2 : ExtraData.js (1037 → 7 modules)

| Fichier créé | Lignes | Responsabilité |
|--------------|--------|----------------|
| `Tooth.js` | 223 | Classe métier allégée |
| `ToothVisualBuilder.js` | 93 | Création sphères, flèches, barycentre |
| `ToothAxesCalculator.js` | 65 | `computeToothMatrix`, `computeToothAxes` |
| `ToothQueryService.js` | 100 | Getters unifiés maloc/world |
| `ToothMovementTracker.js` | 80 | `updateAllMovementsDatas`, `getMovementValue` |
| `ToothStateDispatchers.js` | 45 | Dispatch Redux (player, denied services) |
| `ToothFactory.js` | 200 | `getExtraData`, `initColliders` |
| **ExtraData.js** | **SUPPRIMÉ** | |

**Économie** : 1037 → ~806 lignes (~22% de réduction, mais modularité ×10)

### 11.3 Phase 3 : SceneManager.js (359 → 6 modules + orchestrateur)

| Fichier créé | Lignes | Responsabilité |
|--------------|--------|----------------|
| `SceneInitializer.js` | 102 | Init renderer, lighting, caméra, snapshot, comparative |
| `RenderLoop.js` | 56 | Boucle de rendu multi-viewport |
| `AnimationManager.js` | 23 | État animation (mixer + keyframe) |
| `VisibilityManager.js` | 63 | Visibilité marks/strippings/attachments |
| `ControlSyncManager.js` | 53 | Synchro OrbitControls |
| `ReduxActionCreators.js` | 31 | Création actions Redux |
| **SceneManager.js (nouveau)** | **20** | **Orchestrateur** |

**Économie** : 359 → ~348 lignes (~3% de réduction, mais modularité ×6)

---

## 12. Points d'amélioration

### 12.1 Court terme

1. **Split ThreeEngine** : Le singleton fait 342 lignes. Il pourrait être divisé en :
   - `RendererEngine` (renderer, composer, post-processing)
   - `CameraEngine` (caméras, controls, viewport)
   - `AnimationEngine` (mixer, action, clock)

2. **Tests unitaires** : Aucun test n'est visible dans le codebase. Les modules refactorisés sont maintenant testables unitairement.

3. **TypeScript** : Migration progressive pour typer les entités (`Tooth`, les reducers, les actions).

### 12.2 Moyen terme

4. **React Context vs Redux** : Pour l'état local UI (ex: `isExpanded` du LeftPanel), utiliser React Context au lieu de Redux.

5. **Web Workers** : Le calcul des collisions (Rapier) pourrait être déporté dans un Web Worker.

6. **Virtualization** : Le Player avec beaucoup de keyframes pourrait utiliser la virtualization.

### 12.3 Long terme

7. **Monorepo** : Séparer le moteur 3D (`Domain/`) de l'UI (`Ui/`) en packages indépendants.

8. **WebGPU** : Migrer de WebGL à WebGPU pour de meilleures performances.

9. **State Machine explicite** : Remplacer les booléens épars par une vraie state machine (XState).

---

## Conclusion

L'architecture du Viewer est une **DDD-lite** bien structurée avec une séparation claire entre les couches. Les récentes refactorisations (MeshSelectorEvent, ExtraData, SceneManager) ont considérablement amélioré la maintenabilité en appliquant les principes **GRASP** (Information Expert, Creator, Controller, Low Coupling, High Cohesion).

Les patterns clés identifiés sont : **Singleton**, **Factory**, **Facade**, **Observer**, **Command**, **Module Pattern**.

Les prochaines étapes recommandées sont le split de `ThreeEngine`, l'ajout de tests unitaires, et une migration progressive vers TypeScript.

---

*Fin du rapport*

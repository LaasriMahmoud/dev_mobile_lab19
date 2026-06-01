# 🗄️ Lab 19 : Room, MVVM, Repository, ViewModel, LiveData et RecyclerView

## 📋 Présentation du Projet
Ce laboratoire met en œuvre l'architecture complète **MVVM (Model-View-ViewModel)** recommandée par Google pour le développement Android. L'application **RoomMVVMDemo** permet de gérer une liste de notes avec une persistance de données locale robuste grâce à **Room Database** (une couche d'abstraction sur SQLite).

---

## 🎯 Compétences Visées
- Comprendre la structuration d'une application selon le pattern d'architecture **MVVM**.
- Séparer proprement les responsabilités dans différents packages (`data.local`, `data`, `ui`, `viewmodel`).
- Déclarer des entités de base de données avec des annotations Room (`@Entity`, `@PrimaryKey`).
- Écrire des requêtes d'accès aux données grâce aux interfaces **DAO** (`@Insert`, `@Delete`, `@Query`).
- Implémenter le pattern **Repository** pour centraliser l'accès aux données locales (et éventuellement préparer l'intégration future de sources distantes).
- Utiliser un **ExecutorService** pour déporter les traitements bloquants de base de données en dehors du thread principal (UI Thread).
- Lier les données réactives de la base à l'interface graphique à l'aide de **LiveData** et **RecyclerView**.

---

## 🏗️ Architecture Globale du Projet

Le projet est structuré comme suit :

```
com.example.roommvvmdemo
│
├── data
│   ├── local
│   │   ├── Note.java         <- Entity (Structure SQLite de la table)
│   │   ├── NoteDao.java      <- Data Access Object (Opérations SQL)
│   │   └── NoteDatabase.java <- Instance Singleton de la base de données Room
│   └── NoteRepository.java   <- Couche d'accès intermédiaire aux données
│
├── ui
│   ├── MainActivity.java     <- Vue principale de l'interface
│   └── NoteAdapter.java      <- Gestionnaire d'affichage de la liste RecyclerView
│
└── viewmodel
    └── NoteViewModel.java    <- Conserve l'état de l'UI et communique avec le Repository
```

---

## ⚙️ Détails Techniques Clés

### 1. Pourquoi ExecutorService ?
Room interdit d'effectuer des opérations lourdes d'écriture (Insert/Delete/Update) sur le thread principal pour éviter que l'application ne freeze ou ne crash avec une exception `IllegalStateException`. L'utilisation de `Executors.newSingleThreadExecutor()` dans `NoteRepository` garantit que toutes les écritures s'exécutent en arrière-plan.

### 2. Le couplage puissant : Room + LiveData
La méthode `getAllNotes()` du DAO renvoie un `LiveData<List<Note>>`. Room observe intelligemment la table de la base de données SQLite : dès qu'une note est ajoutée ou supprimée, Room recalcule et réémet automatiquement la liste mise à jour à travers le flux `LiveData`. L'Activity reçoit immédiatement cette mise à jour et rafraîchit l'interface sans requêtage manuel !

---

## 🧪 Scénarios de Test et Validation
1. **Insertion Simple** : Saisir un titre et une description -> Cliquer sur "Ajouter" -> La note apparaît instantanément dans la liste.
2. **Suppression (Clic Long)** : Effectuer un appui prolongé sur une ligne -> La note s'efface de l'interface et de la base avec un message Toast.
3. **Persistance Locale** : Fermer complètement l'application -> La relancer -> Les données sont toujours là.
4. **Rotation d'Écran** : Ajouter des notes -> Tourner l'écran -> Les données sont conservées de manière transparente grâce au `NoteViewModel`.
5. **Suppression Globale** : Cliquer sur "Supprimer toutes les notes" -> La liste se vide entièrement.
# Game.java
1. Utiliser des collections génériques
   Actuellement, les collections telles que ArrayList et LinkedList ne sont pas typées (génériques), ce qui peut engendrer des accidents lors de l'ajout ou la récupération d'éléments.
   Actuel :
   ArrayList players = new ArrayList();
   LinkedList popQuestions = new LinkedList();

   Proposé :
   ArrayList<String> players = new ArrayList<>();
   LinkedList<String> popQuestions = new LinkedList<>();

2. Encapsulation des joueurs dans une classe dédiée
   Les informations des joueurs (emplacement, score, statut) sont gérées par plusieurs tableaux indépendants (e.g., places, purses, inPenaltyBox). Cela rend le code complexe à maintenir et sujet à des erreurs car ces tableaux dépendent de leur index.
   Il serait judicieux de créer une classe Player qui regroupe toutes les données relatives à un joueur.
   Proposé :
   public class Player {
   private String name;
   private int position;
   private int score;
   private boolean inPenaltyBox;

   // Constructeur et méthodes getters/setters
   }
   Puis, utilisez une liste des joueurs :
   ArrayList<Player> players = new ArrayList<>();

3. Supprimer les constantes magiques
   Plusieurs constantes magiques (nombres "hardcodés") rendent le code difficile à maintenir, par exemple :
   La taille du plateau (12 cases).
   Les conditions de victoire (6 pièces d'or).
   Les positions spécifiques associées aux catégories de questions (Pop, Science, Sports).
   Il serait préférable d'utiliser des constantes ou des enums pour ces valeurs.
   Proposé :
   private static final int BOARD_SIZE = 12;
   private static final int WINNING_SCORE = 6;

private enum Category { POP, SCIENCE, SPORTS, ROCK; }

4. Utiliser un Map pour stocker les questions par catégorie
   Plutôt que d'utiliser une collection distincte pour chaque type de question (popQuestions, scienceQuestions, etc.), il est préférable d'utiliser une structure de données comme Map.
   Actuel :
   LinkedList popQuestions = new LinkedList();
   LinkedList scienceQuestions = new LinkedList();
   LinkedList sportsQuestions = new LinkedList();
   LinkedList rockQuestions = new LinkedList();
   Proposé :
   private Map<Category, LinkedList<String>> questions = new HashMap<>();
   Initialisation :
   questions.put(Category.POP, new LinkedList<>());
   questions.put(Category.SCIENCE, new LinkedList<>());
   // etc.
   Utilisation :
   questions.get(currentCategory()).removeFirst();

5. Refactorisation de currentCategory()
   La méthode actuelle utilise de multiples if pour déterminer la catégorie en fonction de la position. Cela rend le code verbeux et moins lisible. Une alternative consiste à stocker ces informations dans une structure plus claire, comme un tableau ou un Map.
   Proposé :
   private String currentCategory() {
   int position = places[currentPlayer] % BOARD_SIZE;
   switch (position) {
   case 0:
   case 4:
   case 8:
   return "Pop";
   case 1:
   case 5:
   case 9:
   return "Science";
   case 2:
   case 6:
   case 10:
   return "Sports";
   default:
   return "Rock";
   }
   }

6. Gérer les erreurs de manière plus robuste
   Votre code suppose que tout se passe parfaitement (par exemple, il y a toujours des questions à poser), mais que se passe-t-il si une liste de questions est vide ? Actuellement, cela entraînerait une exception.
   Ajoutez une gestion des erreurs pour ces cas :
   Proposé :
   private void askQuestion() {
   LinkedList<String> questionList = questions.get(currentCategory());
   if (questionList != null && !questionList.isEmpty()) {
   System.out.println(questionList.removeFirst());
   } else {
   System.out.println("No more questions available for this category!");
   }
   }


7. Ajouter des tests unitaires
   Pour garantir que chaque partie du jeu fonctionne correctement, ajoutez des tests utilisant une bibliothèque comme JUnit. Par exemple :
   Tester la méthode currentCategory().
   Tester l'ajout de joueurs.
   Tester le déroulement d'un tour.
   Exemple (JUnit 5) :
   @Test
   void testCurrentCategory() {
   Game game = new Game();
   game.places[0] = 0;
   assertEquals("Pop", game.currentCategory());

   game.places[0] = 1;
   assertEquals("Science", game.currentCategory());
   }

8. Ajouter des commentaires et respecter les conventions de Java
   Certaines parties du code manquent de commentaires, ce qui peut troubler les développeurs qui lisent le code. Ajoutez des commentaires descriptifs lorsque nécessaire et respectez les conventions Java (par exemple, noms de variables et méthodes en camelCase).

# GameRunner.java

1. Modularité accrue et réduction de la duplication
   La méthode playGame mélange configurations du jeu, logique de boucle, et décisions basées sur les résultats des actions. On peut refactorer cela pour diviser la méthode en sous-méthodes :
   Proposition :
   Ajouter une méthode distincte pour initialiser les joueurs (initializeGame).
   Créer une méthode spécifique pour gérer un tour (playRound).
   Code refactoré :
   public static void main(String[] args) {
   Random randomGenerator = new Random();
   Game game = initializeGame();
   playGame(game, randomGenerator);
   }

private static Game initializeGame() {
Game game = new Game();
game.add("Chet");
game.add("Pat");
game.add("Sue");
return game;
}

private static void playGame(Game game, Random randomGenerator) {
do {
notAWinner = playRound(game, randomGenerator);
} while (notAWinner);
}

private static boolean playRound(Game game, Random randomGenerator) {
game.roll(randomGenerator.nextInt(5) + 1);
if (randomGenerator.nextInt(9) == 7) {
return game.wrongAnswer();
} else {
return game.wasCorrectlyAnswered();
}
}

Avantages :
Le code est découpé en méthodes plus petites et spécialisées.
initializeGame s'occupe uniquement de configurer le jeu.
playRound encapsule la logique d'un tour individuel.
La méthode principale playGame devient plus concise et facile à comprendre.


5. Ajout d’une validation des joueurs
   Le nombre de joueurs doit respecter un minimum (par exemple, 2 joueurs minimum). Nous pouvons ajouter une validation dans la méthode initializeGame.
   Code avec validation :
   private static void validatePlayers(Game game) {
   if (game.numberOfPlayers() < 2) {
   throw new IllegalArgumentException("Le jeu nécessite au moins 2 joueurs !");
   }
   }
   Appelez validatePlayers après l’ajout de tous les joueurs :
   private static Game initializeGame() {
   Game game = new Game();
   game.add("Chet");
   game.add("Pat");
   game.add("Sue");
   validatePlayers(game);
   return game;
   }

6.  Ajout de tests unitaires
   Le jeu peut être testé en se concentrant sur :
   Les règles de base : Si le jeu continue correctement selon les conditions.
   Les générations aléatoires : Si un dé retourne une valeur valide (entre 1 et 6).
   Les cas extrêmes : Si le jeu fonctionne correctement avec un minimum ou un maximum de joueurs.
   Exemple avec JUnit :
   @Test
   public void testGameWithRandomSeed() {
   Game game = initializeGame();
   Random random = new Random(42); // Random fixe pour prédire les comportements.
   GameRunner.playGame(game, random);
   assertTrue(game.isGameOver());
   }


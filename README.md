<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Le Spectre des Émotions de Léia</title>
    <style>
        body {
            font-family: sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f0f0f0;
        }

        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 20px;
        }

        p {
            text-align: center;
            margin-bottom: 30px;
        }

        .emotion-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 20px;
            margin-bottom: 30px;
        }

        .emotion-circle {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            font-size: 12px;
            cursor: pointer;
            border: 2px solid transparent;
        }

        .emotion-circle.selected {
            border: 2px solid black;
        }

        .emotion-circle:hover {
            opacity: 0.8;
        }

        .validation-button-container {
            text-align: center;
            margin-bottom: 20px;
        }

        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
        }

        #confirmation-message {
            text-align: center;
            font-style: italic;
            color: green;
            margin-bottom: 20px;
        }

        .timeline {
            margin-top: 40px;
            border-top: 2px solid #ccc;
            padding-top: 20px;
        }

        .timeline h2 {
            text-align: center;
            margin-bottom: 20px;
        }

        .timeline-graph {
            width: 100%;
            height: 200px;
            background-color: #f9f9f9;
            margin-bottom: 20px;
        }

        .timeline-details {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .timeline-entry {
            border-bottom: 1px solid #eee;
            padding-bottom: 10px;
        }

        #session-id {
            text-align: center;
            margin-top: 20px;
            font-size: 0.8em;
            color: #777;
        }
    </style>
</head>
<body>

    <h1>Création de Nathan permettant d'explorer itérativement le parcours émotionnel de Léia. Cliquer sur une étape permet d'en découvrir les facettes. Cette page interactive doit être complétée au cours d'une journée selon l'évolution émotionnelle (Dans l'idéal 3 fois par jour, au réveil, milieu de journée puis avant de dormir)</h1>

    <div class="emotion-container">
        <div class="emotion-circle" data-emotion="joie" style="background-color: #ffff00;">Joie</div>
        <div class="emotion-circle" data-emotion="tristesse" style="background-color: #0000ff;">Tristesse</div>
        <div class="emotion-circle" data-emotion="colere" style="background-color: #ff0000;">Colère</div>
        <div class="emotion-circle" data-emotion="peur" style="background-color: #800080;">Peur</div>
        <div class="emotion-circle" data-emotion="surprise" style="background-color: #ffa500;">Surprise</div>
        <div class="emotion-circle" data-emotion="degout" style="background-color: #008000;">Dégoût</div>
    </div>

    <div class="validation-button-container">
        <button id="validate-button">Valider mon humeur actuelle</button>
    </div>

    <p id="confirmation-message"></p>

    <div class="timeline">
        <h2>Suivi des Émotions de Léia</h2>
        <div class="timeline-graph">
            </div>
        <div class="timeline-details">
            </div>
    </div>

    <p id="session-id"></p>

    <script>
        const emotionCircles = document.querySelectorAll('.emotion-circle');
        let selectedEmotion = null;

        emotionCircles.forEach(circle => {
            circle.addEventListener('click', () => {
                emotionCircles.forEach(c => c.classList.remove('selected'));
                circle.classList.add('selected');
                selectedEmotion = circle.dataset.emotion;
            });
        });

        const validateButton = document.getElementById('validate-button');
        const confirmationMessage = document.getElementById('confirmation-message');
        const sessionIdDisplay = document.getElementById('session-id');

        // Fonction pour générer un ID de session unique (simplifié)
        function generateSessionId() {
            return Math.random().toString(36).substring(2, 15);
        }

        // ID de session
        let sessionId = generateSessionId();
        sessionIdDisplay.textContent = "Votre ID de session actuel : " + sessionId;

        validateButton.addEventListener('click', () => {
            if (selectedEmotion) {
                // Enregistrement dans Firebase (à remplacer avec votre configuration)
                console.log('Emotion enregistrée:', selectedEmotion, 'Session ID:', sessionId);
                confirmationMessage.textContent = 'Humeur enregistrée !';
                // Réinitialiser la sélection
                emotionCircles.forEach(c => c.classList.remove('selected'));
                selectedEmotion = null;
            } else {
                confirmationMessage.textContent = 'Veuillez sélectionner une humeur.';
            }
        });

        // Fonction pour récupérer et afficher l'historique (à implémenter avec Firebase)
        function fetchAndDisplayHistory(userSessionId = null) {
            // ...
        }

        // Appel initial pour afficher l'historique
        fetchAndDisplayHistory();
    </script>
</body>
</html>

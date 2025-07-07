<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Spectre des Émotions de Léia</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Firebase CDN -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, addDoc, collection, query, where, getDocs, orderBy, limit } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        window.firebase = {
            initializeApp,
            getAuth,
            signInAnonymously,
            signInWithCustomToken,
            onAuthStateChanged,
            getFirestore,
            doc,
            addDoc,
            collection,
            query,
            where,
            getDocs,
            orderBy,
            limit
        };
    </script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutrals -->
    <!-- Application Structure Plan: A single-page application with a two-column layout. The left column features a prominent, vertical, interactive stepper/navigator representing the emotional progression from low to high. Clicking a state on this navigator dynamically updates the right column, which displays detailed information (description, thoughts, behaviors, energy). This interactive navigation provides a clear visual metaphor for the journey through emotional states. Below this main section, a radar chart offers a comparative visualization of the 'energy level' for all five states, providing an at-a-glance summary. A new section is added at the bottom for data tracking and consultation, allowing users to see their current session ID and to retrieve historical selections by other user IDs. This section now includes a line chart to visualize the emotional journey over time and a placeholder for insights/advice, emphasizing public accessibility for collaborative emotional path understanding. -->
    <!-- Visualization & Content Choices: 
        - Report Info: 5 emotional states with descriptions, thoughts, behaviors, energy. Goal: Inform & Compare. Viz: Interactive vertical stepper (HTML/CSS/JS) and dynamic text panels. Interaction: User clicks a state to update content and automatically save selection to Firestore. Justification: Provides a clear, intuitive navigation path that mirrors the emotional journey, with data capture on interaction.
        - Report Info: Energy levels for each state. Goal: Compare. Viz: Radar Chart (Chart.js/Canvas). Interaction: Static visualization with tooltips on hover. Justification: Offers a holistic, visual comparison of a key quantitative metric across all states, reinforcing the concept of progression.
        - Report Info: All textual content. Goal: Inform. Viz: Structured text blocks. Interaction: Dynamically displayed based on user selection. Justification: Keeps the UI clean and focused on one state at a time, preventing information overload.
        - Report Info: User selections over time. Goal: Track & Share. Viz: Textual list/table (HTML/JS) and Line Chart (Chart.js/Canvas). Interaction: Input user ID to filter and display historical data and visualize trends. Justification: Enables the core sharing requirement for tracking Léia's emotional journey and provides visual insight into progression over time.
        - New Feature: Basic "Conseils" section. Goal: Provide potential guidance. Viz: Text block. Interaction: Placeholder for future dynamic advice based on data. Justification: Addresses the user's request for guiding the path towards a better emotional state, laying groundwork for future enhancements.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 550px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 40vh;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 400px;
            }
        }
        .nav-item::after {
            content: '';
            position: absolute;
            left: 1rem;
            top: 2.5rem;
            bottom: -2.5rem;
            width: 2px;
            background-color: #d6d3d1; /* stone-300 */
            z-index: 1;
        }
        .nav-item:last-child::after {
            display: none;
        }
        .nav-item .dot {
            transition: all 0.3s ease;
        }
        .nav-item.active .dot {
            transform: scale(1.5);
            background-color: #f59e0b; /* amber-500 */
            border-color: #f59e0b; /* amber-500 */
        }
        .nav-item.active .nav-text {
            color: #d97706; /* amber-600 */
            font-weight: 700;
        }
        .fade-in {
            animation: fadeIn 0.5s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body class="bg-stone-50 text-stone-800">

    <div class="container mx-auto px-4 sm:px-6 lg:px-8 py-12">
        <header class="text-center mb-12">
            <h1 class="text-4xl md:text-5xl font-bold text-stone-900">Le Spectre des Émotions de Léia</h1>
            <p class="mt-4 text-lg text-stone-600 max-w-3xl mx-auto">Création de Nathan permettant d'explorer itérativement le parcours émotionnel de Léia. Cliquer sur une étape permet d'en découvrir les facettes. Cette page interactive doit être complétée au cours d'une journée selon l'évolution émotionnelle (Dans l'idéal 3 fois par jour, au réveil, milieu de journée puis avant de dormir)</p>
        </header>

        <main class="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-4 gap-12">
            
            <!-- Vertical Stepper Navigation -->
            <aside class="md:col-span-1 lg:col-span-1">
                <nav id="emotion-nav" class="flex md:flex-col justify-between md:justify-start">
                </nav>
            </aside>

            <!-- Content Display -->
            <section class="md:col-span-2 lg:col-span-3 bg-white p-8 rounded-2xl shadow-lg border border-stone-200 min-h-[30rem]">
                <div id="content-display" class="fade-in">
                    <h2 id="state-title" class="text-3xl font-bold text-amber-600 mb-4"></h2>
                    <p id="state-description" class="text-lg text-stone-700 mb-6 italic"></p>
                    
                    <div class="space-y-6">
                        <div>
                            <h3 class="font-bold text-stone-800 text-xl mb-2">Pensées typiques</h3>
                            <ul id="state-thoughts" class="list-none space-y-2">
                            </ul>
                        </div>
                        <div>
                            <h3 class="font-bold text-stone-800 text-xl mb-2">Comportements</h3>
                            <ul id="state-behaviors" class="list-none space-y-2">
                            </ul>
                        </div>
                        <div>
                            <h3 class="font-bold text-stone-800 text-xl mb-2">Niveau d'énergie</h3>
                            <div id="state-energy" class="flex items-center gap-3">
                            </div>
                        </div>
                    </div>
                </div>
            </section>
        </main>
        
        <section class="mt-20 bg-white p-8 rounded-2xl shadow-lg border border-stone-200">
            <header class="text-center mb-8">
                <h2 class="text-3xl font-bold text-stone-900">Comparaison des Niveaux d'Énergie</h2>
                <p class="mt-2 text-md text-stone-600">Ce graphique radar illustre la variation du niveau d'énergie à travers les cinq états émotionnels.</p>
            </header>
            <div class="chart-container">
                <canvas id="energyChart"></canvas>
            </div>
        </section>

        <section class="mt-20 bg-white p-8 rounded-2xl shadow-lg border border-stone-200">
            <header class="text-center mb-8">
                <h2 class="text-3xl font-bold text-stone-900">Suivi Public des Émotions de Léia</h2>
                <p class="mt-2 text-md text-stone-600">Partagez votre ID de session ci-dessous pour permettre à Nathan (ou à toute autre personne) de consulter votre parcours émotionnel. Entrez un ID de session pour visualiser l'historique et les tendances.</p>
            </header>
            <div class="max-w-xl mx-auto space-y-6">
                <div class="p-4 bg-stone-100 rounded-lg text-center">
                    <p class="text-lg font-semibold text-stone-700">Votre ID de session actuel :</p>
                    <p id="current-user-id" class="text-xl font-bold text-amber-700 break-all mt-2">Chargement...</p>
                </div>

                <div class="flex flex-col sm:flex-row gap-4 items-center">
                    <input type="text" id="consult-user-id-input" placeholder="Entrez un ID de session à consulter" class="flex-grow p-3 border border-stone-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-amber-500">
                    <button id="consult-button" class="bg-amber-600 hover:bg-amber-700 text-white font-bold py-3 px-6 rounded-lg shadow-md transition-colors duration-300 w-full sm:w-auto">
                        Consulter les sélections
                    </button>
                </div>
                
                <div id="selection-history-container" class="mt-6 border border-stone-200 rounded-lg p-4 bg-stone-50">
                    <p id="history-message" class="text-stone-500 text-center mb-4">Aucune sélection consultée pour le moment.</p>
                    <div class="chart-container mb-6">
                        <canvas id="historyLineChart"></canvas>
                    </div>
                    <h3 class="text-xl font-bold text-stone-800 mb-2">Historique détaillé :</h3>
                    <ul id="history-list" class="space-y-2 max-h-60 overflow-y-auto">
                    </ul>
                    <h3 class="text-xl font-bold text-stone-800 mt-6 mb-2">Conseils :</h3>
                    <p id="advice-text" class="text-stone-700">En observant votre parcours émotionnel, vous pourrez identifier des tendances. Par exemple, si vous passez souvent d'un état d'exaltation à une tristesse profonde, cela pourrait indiquer un besoin de stabilité. Si vous restez longtemps dans la résignation, cherchez des activités qui vous apportent de petites joies. Le chemin vers un meilleur état émotionnel est unique à chacun et se construit pas à pas.</p>
                </div>
            </div>
        </section>

    </div>

    <script type="module">
        const emotionData = [
            {
                id: 0,
                title: "Moment de tristesse profonde",
                description: "État de désespoir, de douleur intense et de perte d'intérêt pour toute activité. Sensation d'accablement, d'isolement et de vide.",
                thoughts: ["Rien ne va s'améliorer.", "Je suis seule.", "C'est insurmontable."],
                behaviors: ["Retrait social", "Léthargie", "Pleurs", "Difficulté à se concentrer"],
                energy: "Très faible",
                energyValue: 1
            },
            {
                id: 1,
                title: "État de résignation/apathie",
                description: "Une forme de tristesse moins aiguë, mais caractérisée par un manque d'émotion, d'énergie et de motivation. On accepte la situation sans chercher à la changer.",
                thoughts: ["C'est comme ça.", "À quoi bon ?", "Je n'ai pas la force."],
                behaviors: ["Passivité", "Évitement", "Routine minimale"],
                energy: "Faible à modérée",
                energyValue: 2
            },
            {
                id: 2,
                title: "Phase de neutralité/calme",
                description: "Un état d'équilibre où les émotions intenses sont absentes. C'est un moment de repos émotionnel, de réflexion ou de simple existence.",
                thoughts: ["Je suis calme.", "Je me sens stable."],
                behaviors: ["Activités quotidiennes normales", "Observation", "Planification potentielle"],
                energy: "Modérée",
                energyValue: 3
            },
            {
                id: 3,
                title: "Sentiment d'optimisme/espoir",
                description: "Une émotion positive naissante, caractérisée par la conviction que les choses peuvent s'améliorer. Un regain d'énergie et une ouverture aux possibilités futures.",
                thoughts: ["Peut-être que ça va aller.", "Il y a des solutions.", "Je peux essayer."],
                behaviors: ["Engagement progressif", "Recherche de solutions", "Interaction positive"],
                energy: "Moyenne à élevée",
                energyValue: 4
            },
            {
                id: 4,
                title: "Moment d'exaltation",
                description: "État de joie intense, d'enthousiasme, de confiance en soi et d'énergie débordante. Sensation de puissance, d'accomplissement et de connexion.",
                thoughts: ["Je peux tout faire !", "C'est merveilleux !", "La vie est belle !"],
                behaviors: ["Proactivité", "Créativité", "Célébration", "Partage de l'énergie"],
                energy: "Très élevée",
                energyValue: 5
            }
        ];

        const navContainer = document.getElementById('emotion-nav');
        const contentDisplay = document.getElementById('content-display');
        const stateTitle = document.getElementById('state-title');
        const stateDescription = document.getElementById('state-description');
        const stateThoughts = document.getElementById('state-thoughts');
        const stateBehaviors = document.getElementById('state-behaviors');
        const stateEnergy = document.getElementById('state-energy');
        const currentUserIdDisplay = document.getElementById('current-user-id');
        const consultUserIdInput = document.getElementById('consult-user-id-input');
        const consultButton = document.getElementById('consult-button');
        const historyList = document.getElementById('history-list');
        const historyMessage = document.getElementById('history-message');
        const historyLineChartCanvas = document.getElementById('historyLineChart');

        let activeStateId = 0;
        let db, auth, currentUserId;
        let isAuthReady = false;
        let lineChartInstance = null; // To store the Chart.js instance for the line chart

        // Initialize Firebase
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};

        try {
            const app = window.firebase.initializeApp(firebaseConfig);
            db = window.firebase.getFirestore(app);
            auth = window.firebase.getAuth(app);

            window.firebase.onAuthStateChanged(auth, async (user) => {
                if (user) {
                    currentUserId = user.uid;
                    currentUserIdDisplay.textContent = currentUserId;
                    isAuthReady = true;
                } else {
                    try {
                        const credential = await window.firebase.signInAnonymously(auth);
                        currentUserId = credential.user.uid;
                        currentUserIdDisplay.textContent = currentUserId;
                        isAuthReady = true;
                    } catch (error) {
                        console.error("Erreur d'authentification anonyme :", error);
                        currentUserIdDisplay.textContent = "Erreur de chargement de l'ID";
                    }
                }
            });
        } catch (error) {
            console.error("Erreur d'initialisation de Firebase :", error);
            currentUserIdDisplay.textContent = "Firebase non initialisé";
        }

        function createEnergyBar(value, text) {
            const container = document.createElement('div');
            container.className = 'w-full bg-stone-200 rounded-full h-2.5';
            const bar = document.createElement('div');
            const percentage = (value / 5) * 100;
            const colors = ['bg-red-400', 'bg-orange-400', 'bg-yellow-400', 'bg-lime-400', 'bg-green-400'];
            bar.className = `${colors[value-1]} h-2.5 rounded-full transition-all duration-500`;
            bar.style.width = `${percentage}%`;
            container.appendChild(bar);
            
            const textEl = document.createElement('span');
            textEl.className = 'text-stone-600 font-medium';
            textEl.textContent = text;
            
            const wrapper = document.createElement('div');
            wrapper.className = 'flex items-center gap-4 w-full';
            const barWrapper = document.createElement('div');
            barWrapper.className = 'w-1/2';
            barWrapper.appendChild(container);
            
            wrapper.appendChild(textEl);
            wrapper.appendChild(barWrapper);
            
            return wrapper;
        }

        async function saveSelection(stateId) {
            if (!isAuthReady || !db || !currentUserId) {
                console.warn("Firebase non prêt ou utilisateur non authentifié pour sauvegarder la sélection.");
                return;
            }
            try {
                const selectionsRef = window.firebase.collection(db, `artifacts/${appId}/public/data/leia_emotions_tracking`);
                await window.firebase.addDoc(selectionsRef, {
                    userId: currentUserId,
                    stateId: stateId,
                    timestamp: new Date().toISOString()
                });
                console.log("Sélection sauvegardée avec succès !");
            } catch (e) {
                console.error("Erreur lors de l'ajout du document : ", e);
            }
        }

        function updateContent(stateId) {
            const state = emotionData.find(s => s.id === stateId);
            if (!state) return;

            contentDisplay.classList.remove('fade-in');
            void contentDisplay.offsetWidth;
            contentDisplay.classList.add('fade-in');

            stateTitle.textContent = state.title;
            stateDescription.textContent = state.description;

            stateThoughts.innerHTML = state.thoughts.map(thought => 
                `<li class="flex items-start"><span class="text-amber-500 mr-3 mt-1">&#10003;</span><span>${thought}</span></li>`
            ).join('');

            stateBehaviors.innerHTML = state.behaviors.map(behavior => 
                `<li class="flex items-start"><span class="text-amber-500 mr-3 mt-1">&#10148;</span><span>${behavior}</span></li>`
            ).join('');

            stateEnergy.innerHTML = '';
            stateEnergy.appendChild(createEnergyBar(state.energyValue, state.energy));

            document.querySelectorAll('.nav-item').forEach(item => {
                item.classList.toggle('active', parseInt(item.dataset.stateId) === stateId);
            });

            activeStateId = stateId;
            saveSelection(stateId); // Automatic save on click
        }
        
        function setupNavigation() {
            emotionData.forEach(state => {
                const navItem = document.createElement('div');
                navItem.className = 'nav-item relative flex items-center cursor-pointer p-2 md:pl-0 md:pr-4 group';
                navItem.dataset.stateId = state.id;
                
                navItem.innerHTML = `
                    <div class="relative z-10 flex items-center">
                        <div class="dot w-8 h-8 rounded-full border-2 border-stone-300 bg-white flex-shrink-0"></div>
                        <span class="nav-text ml-4 text-stone-600 group-hover:text-stone-900 transition-colors">${state.title}</span>
                    </div>
                `;

                navItem.addEventListener('click', () => {
                    updateContent(state.id);
                });
                navContainer.appendChild(navItem);
            });
        }
        
        function setupChart() {
            const ctx = document.getElementById('energyChart').getContext('2d');
            const labels = emotionData.map(s => s.title.split('/')[0]);
            const data = emotionData.map(s => s.energyValue);

            new Chart(ctx, {
                type: 'radar',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Niveau d\'Énergie',
                        data: data,
                        fill: true,
                        backgroundColor: 'rgba(245, 158, 11, 0.2)',
                        borderColor: 'rgb(245, 158, 11)',
                        pointBackgroundColor: 'rgb(245, 158, 11)',
                        pointBorderColor: '#fff',
                        pointHoverBackgroundColor: '#fff',
                        pointHoverBorderColor: 'rgb(245, 158, 11)'
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        r: {
                            angleLines: {
                                color: 'rgba(120, 113, 108, 0.2)'
                            },
                            grid: {
                                color: 'rgba(120, 113, 108, 0.2)'
                            },
                            pointLabels: {
                                font: {
                                    size: 12,
                                    weight: '500'
                                },
                                color: '#44403c'
                            },
                            ticks: {
                                backdropColor: 'rgba(250, 250, 249, 0)',
                                color: '#78716c',
                                stepSize: 1,
                                font: {
                                    size: 10
                                }
                            },
                            suggestedMin: 0,
                            suggestedMax: 5
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        }
                    }
                }
            });
        }

        function renderLineChart(selections) {
            if (lineChartInstance) {
                lineChartInstance.destroy(); // Destroy previous instance if it exists
            }

            const labels = selections.map(s => new Date(s.timestamp).toLocaleString('fr-FR', { month: 'numeric', day: 'numeric', hour: '2-digit', minute: '2-digit' }));
            const data = selections.map(s => {
                const state = emotionData.find(ed => ed.id === s.stateId);
                return state ? state.energyValue : 0; // Use energyValue for the line chart
            });

            const ctx = historyLineChartCanvas.getContext('2d');
            lineChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Niveau d\'Énergie Émotionnelle',
                        data: data,
                        borderColor: 'rgb(75, 192, 192)',
                        backgroundColor: 'rgba(75, 192, 192, 0.2)',
                        tension: 0.1,
                        fill: false,
                        pointRadius: 5,
                        pointHoverRadius: 8,
                        pointBackgroundColor: 'rgb(75, 192, 192)',
                        pointBorderColor: '#fff'
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: {
                            title: {
                                display: true,
                                text: 'Temps',
                                color: '#44403c'
                            },
                            ticks: {
                                color: '#78716c'
                            }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'Niveau d\'Énergie (1=Très faible, 5=Très élevée)',
                                color: '#44403c'
                            },
                            min: 0,
                            max: 5,
                            ticks: {
                                stepSize: 1,
                                color: '#78716c'
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: true,
                            labels: {
                                color: '#44403c'
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    const energyVal = context.raw;
                                    const state = emotionData.find(ed => ed.energyValue === energyVal);
                                    return label + (state ? state.title : energyVal);
                                }
                            }
                        }
                    }
                }
            });
        }

        async function fetchSelectionsForUser(userIdToConsult) {
            if (!isAuthReady || !db) {
                console.warn("Firebase non prêt pour consulter les sélections.");
                historyMessage.textContent = "Veuillez patienter, Firebase se charge...";
                return;
            }

            historyList.innerHTML = '';
            historyMessage.textContent = "Chargement des sélections...";

            try {
                const selectionsRef = window.firebase.collection(db, `artifacts/${appId}/public/data/leia_emotions_tracking`);
                const q = window.firebase.query(
                    selectionsRef,
                    window.firebase.where("userId", "==", userIdToConsult)
                );
                const querySnapshot = await window.firebase.getDocs(q);

                if (querySnapshot.empty) {
                    historyMessage.textContent = `Aucune sélection trouvée pour l'ID : ${userIdToConsult}`;
                    renderLineChart([]); // Clear the chart if no data
                    return;
                }

                const selections = [];
                querySnapshot.forEach((doc) => {
                    selections.push(doc.data());
                });

                // Sort in memory by timestamp (ascending for chart)
                selections.sort((a, b) => new Date(a.timestamp) - new Date(b.timestamp));

                historyMessage.textContent = `Sélections pour l'ID : ${userIdToConsult}`;
                selections.forEach(selection => {
                    const state = emotionData.find(s => s.id === selection.stateId);
                    const stateName = state ? state.title : `État inconnu (${selection.stateId})`;
                    const date = new Date(selection.timestamp).toLocaleString('fr-FR');
                    const listItem = document.createElement('li');
                    listItem.className = 'p-2 border-b border-stone-100 last:border-b-0 text-stone-700';
                    listItem.textContent = `${date} : ${stateName}`;
                    historyList.appendChild(listItem);
                });

                renderLineChart(selections); // Render the line chart with fetched data

            } catch (error) {
                console.error("Erreur lors de la consultation des sélections :", error);
                historyMessage.textContent = "Erreur lors du chargement de l'historique.";
                renderLineChart([]); // Clear chart on error
            }
        }

        consultButton.addEventListener('click', () => {
            const userIdToConsult = consultUserIdInput.value.trim();
            if (userIdToConsult) {
                fetchSelectionsForUser(userIdToConsult);
            } else {
                historyMessage.textContent = "Veuillez entrer un ID de session.";
            }
        });

        document.addEventListener('DOMContentLoaded', () => {
            setupNavigation();
            updateContent(0); // Display initial state
            setupChart();
            renderLineChart([]); // Initialize empty line chart on load
        });
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Spectre des Émotions de Léia</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutrals -->
    <!-- Application Structure Plan: A single-page application with a two-column layout. The left column features a prominent, vertical, interactive stepper/navigator representing the emotional progression from low to high. Clicking a state on this navigator dynamically updates the right column, which displays detailed information (description, thoughts, behaviors, energy). This interactive navigation provides a clear visual metaphor for the journey through emotional states. Below this main section, a radar chart offers a comparative visualization of the 'energy level' for all five states, providing an at-a-glance summary. This structure was chosen to transform a linear text document into an engaging, explorable experience, allowing users to actively select and compare information rather than passively reading. -->
    <!-- Visualization & Content Choices: 
        - Report Info: 5 emotional states with descriptions, thoughts, behaviors, energy. Goal: Inform & Compare. Viz: Interactive vertical stepper (HTML/CSS/JS) and dynamic text panels. Interaction: User clicks a state to update content. Justification: Provides a clear, intuitive navigation path that mirrors the emotional journey.
        - Report Info: Energy levels for each state. Goal: Compare. Viz: Radar Chart (Chart.js/Canvas). Interaction: Static visualization with tooltips on hover. Justification: Offers a holistic, visual comparison of a key quantitative metric across all states, reinforcing the concept of progression.
        - Report Info: All textual content. Goal: Inform. Viz: Structured text blocks. Interaction: Dynamically displayed based on user selection. Justification: Keeps the UI clean and focused on one state at a time, preventing information overload.
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

    </div>

    <script>
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

        let activeStateId = 0;

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

        document.addEventListener('DOMContentLoaded', () => {
            setupNavigation();
            updateContent(0);
            setupChart();
        });
    </script>
</body>
</html>

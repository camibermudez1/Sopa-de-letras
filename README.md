# Sopa-de-letras
Objetivo: Reforzar el reconocimiento visual, la ortografía y el vocabulario en inglés de los nombres de animales mediante una dinámica lúdica que fomente la atención y concentración.
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sopa de Letras de Animales</title>
    <!-- Tailwind CSS CDN para estilos rápidos y responsivos -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0fdf4; /* Un verde claro para el fondo */
        }
        .word-search-grid {
            display: grid;
            grid-template-columns: repeat(10, 1fr); /* 10 columnas */
            gap: 2px;
            max-width: 500px;
            margin: 0 auto;
            border: 2px solid #34d399; /* Borde verde para la cuadrícula */
            border-radius: 10px;
            overflow: hidden; /* Asegura que los bordes redondeados se vean bien */
        }
        .grid-cell {
            width: 100%;
            padding-top: 100%; /* Truco para hacer las celdas cuadradas */
            position: relative;
            background-color: #ecfdf5; /* Fondo más claro para las celdas */
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            font-size: clamp(1rem, 4vw, 1.5rem); /* Tamaño de fuente responsivo */
            font-weight: 600;
            color: #10b981; /* Color de letra principal */
            user-select: none; /* Evita la selección de texto nativa */
            transition: background-color 0.2s ease, color 0.2s ease;
        }
        .grid-cell.selected {
            background-color: #a7f3d0; /* Color cuando la celda está seleccionada */
            color: #065f46; /* Color de letra cuando está seleccionada */
        }
        .grid-cell.found {
            background-color: #34d399; /* Color cuando la palabra ha sido encontrada */
            color: #ffffff; /* Letra blanca para palabras encontradas */
            /* Se quitó 'pointer-events: none;' para permitir la re-selección de celdas encontradas */
        }
        .grid-cell:hover:not(.found) {
            background-color: #d1fae5; /* Efecto hover */
        }
        .word-list li {
            font-size: clamp(1rem, 3vw, 1.25rem);
            color: #059669; /* Color de la lista de palabras */
            padding: 4px 0;
            transition: text-decoration 0.3s ease, color 0.3s ease;
            display: flex; /* Para alinear el emoji con el texto */
            align-items: center;
            justify-content: center; /* Centrar contenido de la lista */
            gap: 8px; /* Espacio entre texto y emoji */
        }
        .word-list li.found {
            text-decoration: line-through; /* Tachado para palabras encontradas */
            color: #6ee7b7; /* Color más suave para palabras encontradas */
        }
        .message-box {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 20px 30px;
            border-radius: 10px;
            text-align: center;
            font-size: 1.2rem;
            z-index: 1000;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.3s ease, visibility 0.3s ease;
        }
        .message-box.show {
            opacity: 1;
            visibility: visible;
        }
    </style>
</head>
<body class="p-4 bg-green-50 min-h-screen flex items-center justify-center">

    <div class="container mx-auto p-6 bg-white rounded-lg shadow-xl flex flex-col md:flex-row gap-8 max-w-4xl">
        <div class="flex-1">
            <h1 class="text-3xl font-bold text-center text-green-700 mb-4">Actividad 4: Sopa de letras</h1>
            <p class="text-center text-gray-600 mb-6 text-lg">
                Encuentra los nombres de los animales en inglés escondidos en esta sopa de letras.
                Pueden estar en horizontal, vertical o diagonal. ¡Buena suerte!
            </p>

            <div class="word-search-grid">
                <!-- Las celdas de la cuadrícula se generarán con JavaScript -->
            </div>
            <div class="flex flex-col sm:flex-row gap-4 mt-6">
                <button id="resetButton" class="flex-1 py-3 px-6 bg-yellow-500 hover:bg-yellow-600 text-white font-bold rounded-lg shadow-md transition duration-300">
                    Reiniciar Juego
                </button>
                <button id="solveButton" class="flex-1 py-3 px-6 bg-blue-500 hover:bg-blue-600 text-white font-bold rounded-lg shadow-md transition duration-300">
                    Mostrar Solución
                </button>
            </div>
        </div>

        <div class="flex-1 md:w-1/3 p-4 bg-green-50 rounded-lg shadow-inner">
            <h2 class="text-2xl font-semibold text-green-600 mb-4 text-center">Palabras a buscar:</h2>
            <ul id="wordList" class="word-list list-none p-0 text-center">
                <!-- La lista de palabras se generará con JavaScript -->
            </ul>
        </div>
    </div>

    <!-- Message Box for feedback -->
    <div id="messageBox" class="message-box"></div>

    <script>
        // Define las palabras a buscar
        const wordsToFind = ['DOG', 'CAT', 'LION', 'COW', 'BEAR', 'PIG', 'HORSE', 'DUCK', 'FROG', 'SHEEP'];
        let foundWords = []; // Array para almacenar las palabras encontradas

        // Mapeo de palabras a emojis
        const emojis = {
            'DOG': '🐶', 'CAT': '🐱', 'LION': '🦁', 'COW': '🐄', 'BEAR': '🐻',
            'PIG': '🐷', 'HORSE': '🐴', 'DUCK': '🦆', 'FROG': '🐸', 'SHEEP': '🐑'
        };

        // Cuadrícula del rompecabezas (10x10) con todas las palabras incrustadas.
        // Las letras 'X' serán reemplazadas por letras aleatorias al inicializar.
        const puzzleGridTemplate = [
            ['D', 'O', 'G', 'X', 'X', 'L', 'X', 'X', 'X', 'X'], // DOG, LION (V)
            ['H', 'O', 'R', 'S', 'E', 'I', 'X', 'X', 'X', 'X'], // HORSE, LION (V)
            ['B', 'E', 'A', 'R', 'X', 'O', 'X', 'X', 'X', 'X'], // BEAR, LION (V)
            ['X', 'X', 'X', 'X', 'X', 'N', 'X', 'X', 'X', 'X'], // LION (V)
            ['X', 'X', 'X', 'X', 'X', 'X', 'X', 'C', 'O', 'W'], // COW
            ['S', 'H', 'E', 'E', 'P', 'X', 'X', 'X', 'X', 'X'], // SHEEP
            ['X', 'X', 'P', 'I', 'G', 'X', 'X', 'X', 'X', 'X'], // PIG
            ['X', 'X', 'X', 'X', 'X', 'F', 'R', 'O', 'G', 'X'], // FROG
            ['D', 'U', 'C', 'K', 'X', 'X', 'X', 'X', 'X', 'X'], // DUCK
            ['C', 'A', 'T', 'X', 'X', 'X', 'X', 'X', 'X', 'X']  // CAT
        ];

        let puzzleGrid = []; // La cuadrícula final con letras aleatorias

        let selectedCells = []; // Almacena las celdas seleccionadas por el usuario
        const gridContainer = document.querySelector('.word-search-grid');
        const wordListElement = document.getElementById('wordList');
        const messageBox = document.getElementById('messageBox');
        const resetButton = document.getElementById('resetButton');
        const solveButton = document.getElementById('solveButton'); // Nuevo botón de solución

        // Función para generar una letra aleatoria
        function getRandomLetter() {
            const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
            return alphabet[Math.floor(Math.random() * alphabet.length)];
        }

        // Función para mostrar un mensaje temporal
        function showMessage(message, duration = 2000) {
            messageBox.textContent = message;
            messageBox.classList.add('show');
            setTimeout(() => {
                messageBox.classList.remove('show');
            }, duration);
        }

        // Función para inicializar la cuadrícula y la lista de palabras
        function initializeGame() {
            gridContainer.innerHTML = ''; // Limpiar cuadrícula existente
            wordListElement.innerHTML = ''; // Limpiar lista de palabras
            selectedCells = [];
            foundWords = [];

            // Generar la cuadrícula con letras aleatorias para los espacios 'X'
            puzzleGrid = puzzleGridTemplate.map(row =>
                row.map(cell => (cell === 'X' ? getRandomLetter() : cell))
            );

            // Crear las celdas de la cuadrícula
            puzzleGrid.flat().forEach((letter, index) => {
                const cell = document.createElement('div');
                cell.classList.add('grid-cell');
                cell.textContent = letter;
                cell.dataset.index = index; // Guardar el índice plano para referencia
                cell.addEventListener('click', handleCellClick);
                gridContainer.appendChild(cell);
            });

            // Crear la lista de palabras con emojis
            wordsToFind.forEach(word => {
                const li = document.createElement('li');
                li.id = `word-${word}`; // ID para facilitar la actualización de estilo
                li.innerHTML = `${word} <span class="text-2xl">${emojis[word] || ''}</span>`; // Añadir emoji con tamaño específico
                wordListElement.appendChild(li);
            });
        }

        // Manejador de clics en las celdas de la cuadrícula
        function handleCellClick(event) {
            const clickedCell = event.target;

            // Siempre permite alternar la selección
            clickedCell.classList.toggle('selected');

            if (clickedCell.classList.contains('selected')) {
                selectedCells.push(clickedCell);
            } else {
                // Eliminar la celda de las seleccionadas si se deselecciona
                selectedCells = selectedCells.filter(cell => cell !== clickedCell);
            }

            // Ordenar las celdas seleccionadas por su índice para facilitar la verificación
            selectedCells.sort((a, b) => parseInt(a.dataset.index) - parseInt(b.dataset.index));

            // Si hay al menos dos celdas seleccionadas, intentar formar una palabra
            if (selectedCells.length >= 2) {
                checkWord();
            }
        }

        // Función para verificar si las celdas seleccionadas forman una palabra
        function checkWord() {
            if (selectedCells.length === 0) return;

            // Obtener la palabra formada por las celdas seleccionadas
            let formedWord = '';
            selectedCells.forEach(cell => {
                formedWord += cell.textContent;
            });

            // Normalizar la palabra para buscarla (en mayúsculas)
            const normalizedFormedWord = formedWord.toUpperCase();

            // Verificar si la palabra es una de las que buscamos y no ha sido encontrada aún
            if (wordsToFind.includes(normalizedFormedWord) && !foundWords.includes(normalizedFormedWord)) {
                // Verificar si las celdas seleccionadas forman una línea recta (horizontal, vertical o diagonal)
                if (isStraightLine(selectedCells)) {
                    foundWords.push(normalizedFormedWord);
                    markCellsAsFound(selectedCells);
                    updateWordList(normalizedFormedWord);
                    // Actualizar el mensaje de felicitación
                    showMessage(`¡Felicitaciones, encontraste ${normalizedFormedWord}! ¡Sigue así!`, 1500);

                    // Reiniciar la selección después de encontrar una palabra
                    clearSelection();

                    // Comprobar si todas las palabras han sido encontradas
                    if (foundWords.length === wordsToFind.length) {
                        showMessage('¡Felicidades! ¡Has encontrado todas las palabras!', 3000);
                    }
                    return; // Palabra encontrada, salir
                }
            }

            // Si la palabra no es válida o no está en línea recta, limpiar la selección si hay más de 1 celda
            if (selectedCells.length > 1) {
                // Comprobar si la selección actual es un prefijo de alguna palabra a buscar.
                const isPrefix = wordsToFind.some(word => word.startsWith(normalizedFormedWord));

                if (!isPrefix) {
                    showMessage('Inténtalo de nuevo. Esa no es una palabra válida o no está en línea recta.', 1500);
                    clearSelection();
                }
            }
        }

        // Función para verificar si un conjunto de celdas forma una línea recta (horizontal, vertical o diagonal)
        function isStraightLine(cells) {
            if (cells.length < 2) return true;

            const numCols = puzzleGrid[0].length;

            const coords = cells.map(cell => {
                const index = parseInt(cell.dataset.index);
                return {
                    row: Math.floor(index / numCols),
                    col: index % numCols
                };
            });

            coords.sort((a, b) => {
                if (a.row !== b.row) return a.row - b.row;
                return a.col - b.col;
            });

            const first = coords[0];
            const dRow = coords.length > 1 ? coords[1].row - coords[0].row : 0;
            const dCol = coords.length > 1 ? coords[1].col - coords[0].col : 0;

            if ((dRow === 0 && dCol !== 0) ||
                (dRow !== 0 && dCol === 0) ||
                (Math.abs(dRow) === Math.abs(dCol) && dRow !== 0 && dCol !== 0)) {
                for (let i = 1; i < coords.length; i++) {
                    if (coords[i].row !== coords[i - 1].row + dRow ||
                        coords[i].col !== coords[i - 1].col + dCol) {
                        return false;
                    }
                }
                return true;
            }

            return false;
        }

        // Función para marcar las celdas como encontradas
        function markCellsAsFound(cells) {
            cells.forEach(cell => {
                cell.classList.add('found');
            });
        }

        // Función para actualizar el estilo de la palabra en la lista
        function updateWordList(word) {
            const listItem = document.getElementById(`word-${word}`);
            if (listItem) {
                listItem.classList.add('found');
            }
        }

        // Función para limpiar la selección actual de celdas
        function clearSelection() {
            selectedCells.forEach(cell => {
                cell.classList.remove('selected');
            });
            selectedCells = [];
        }

        // --- Nueva función para resolver el puzzle ---
        function solvePuzzle() {
            const numRows = puzzleGrid.length;
            const numCols = puzzleGrid[0].length;
            const allCells = Array.from(document.querySelectorAll('.grid-cell'));

            clearSelection(); // Limpiar cualquier selección actual del usuario

            wordsToFind.forEach(word => {
                const wordLength = word.length;
                let wordFoundInPuzzle = false;

                // Direcciones: [dRow, dCol] (Horizontal, Vertical, Diagonal abajo-derecha, Diagonal abajo-izquierda)
                // Incluye direcciones inversas para cubrir todas las posibilidades
                const directions = [
                    [0, 1],   // Derecha
                    [1, 0],   // Abajo
                    [1, 1],   // Diagonal abajo-derecha
                    [1, -1],  // Diagonal abajo-izquierda
                    [0, -1],  // Izquierda
                    [-1, 0],  // Arriba
                    [-1, -1], // Diagonal arriba-izquierda
                    [-1, 1]   // Diagonal arriba-derecha
                ];

                for (let row = 0; row < numRows; row++) {
                    for (let col = 0; col < numCols; col++) {
                        for (const [dr, dc] of directions) {
                            let currentWord = '';
                            let cellsToMark = [];
                            let validPath = true;

                            for (let k = 0; k < wordLength; k++) {
                                const newRow = row + k * dr;
                                const newCol = col + k * dc;

                                // Comprobar límites
                                if (newRow >= 0 && newRow < numRows && newCol >= 0 && newCol < numCols) {
                                    currentWord += puzzleGrid[newRow][newCol];
                                    const index = newRow * numCols + newCol;
                                    cellsToMark.push(allCells[index]);
                                } else {
                                    validPath = false;
                                    break;
                                }
                            }

                            if (validPath && currentWord === word) {
                                markCellsAsFound(cellsToMark);
                                updateWordList(word);
                                // Asegúrate de que la palabra se añada a foundWords para el estado del juego
                                if (!foundWords.includes(word)) {
                                    foundWords.push(word);
                                }
                                wordFoundInPuzzle = true;
                                break; // Palabra encontrada en esta dirección, pasar a la siguiente palabra
                            }
                        }
                        if (wordFoundInPuzzle) break;
                    }
                    if (wordFoundInPuzzle) break;
                }
            });

            // Mensaje final
            showMessage('¡Todas las palabras han sido marcadas!', 3000);
        }


        // Event listener para el botón de reiniciar
        resetButton.addEventListener('click', initializeGame);
        // Event listener para el nuevo botón de solución
        solveButton.addEventListener('click', solvePuzzle);

        // Inicializar el juego cuando la ventana se cargue
        window.onload = initializeGame;
    </script>
</body>
</html>
Elaborado por: Camila Bermudez
